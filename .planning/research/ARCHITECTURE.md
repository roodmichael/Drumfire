# Architecture Research — Drumfire

**Domain:** Community Coffee Roast Logging SaaS
**Confidence:** HIGH — patterns drawn from stable, well-documented SaaS and social graph conventions

---

## Key Findings

- The existing JSON model maps cleanly to PostgreSQL with one structural change: cuppings should be extracted from embedded arrays into a normalized `cuppings` table to support SQL queries (defect trends, aggregate ratings). The snapshot pattern (frozen JSONB copies of bean + roaster at roast time) is correct and should be preserved as-is.
- Beans should remain **global/community-owned** (not scoped to users). Two roasters logging the same Ethiopian Yirgacheffe should reference the same bean record. This is the architecture that makes bean-search the core differentiator.
- The social feed should use a **pull fan-in query** at launch — a single SQL JOIN against the `follows` table with a covering index is fast enough for thousands of users. Fan-out pre-computation adds Redis + background job complexity before it is needed.
- Live logging should use **REST for writes, WebSocket for broadcast only**. Roast events are durable state — they must go through the API to PostgreSQL, not through WebSocket messages. WebSocket is a display layer for observers.
- Build order has a clean dependency chain: schema → auth → beans/roasters → roasts → cuppings → social graph → bean search → live logging. The product is shippable after step 5 (complete single-user logging app), with the community layer added in steps 6-7.

---

## Component Boundaries

| Component | Responsibility | Communicates With |
|-----------|---------------|-------------------|
| Frontend (Next.js App Router) | UI, routing, auth token storage, live logging timer | API via Server Actions + REST + WebSocket |
| API Server (stateless) | Auth, business logic, query construction, access control | Frontend, PostgreSQL, Redis |
| PostgreSQL | Canonical data store, full-text search | API Server |
| Redis (optional at launch) | Feed cache, WebSocket pubsub for live logging, rate limiting | API Server |
| Object Storage (S3-compatible) | Profile images only | API Server (presigned URLs) |

Redis is not required for launch. The system is correct without it. Add it when feed queries measure slow or when the app scales past a single web server instance.

---

## Data Flow

**Write path (roast creation):**
```
POST /roasts → validate → INSERT roasts + roast_adjustments (transaction) → optional: enqueue feed fan-out → return record
```

**Read path (bean search):**
```
GET /beans?q=ethiopia → full-text GIN index query → return beans with public roast counts
User selects bean → GET /roasts?bean_id=X&visibility=public → paginated roast list
```

**Feed path (pull model):**
```sql
SELECT r.*, u.username FROM roasts r
JOIN users u ON r.user_id = u.id
WHERE r.user_id IN (SELECT followee_id FROM follows WHERE follower_id = $current_user)
AND r.visibility = 'public'
ORDER BY r.created_at DESC LIMIT 20
```
Index: `CREATE INDEX roasts_user_created_idx ON roasts(user_id, created_at DESC)`

**Live logging path:**
```
POST /roasts → in_progress status → client starts local timer
User taps event → PATCH /roasts/:id (REST, durable) → server broadcasts event to WebSocket room
Observers receive event, update display
```

---

## Core Schema (PostgreSQL)

```sql
-- Users
users (id uuid PK, email text UNIQUE, username text UNIQUE, display_name, bio, avatar_url, created_at)

-- Equipment (user-scoped)
roasters (id uuid PK, user_id uuid FK, name, manufacturer, model, type,
          capacity_min_g, capacity_max_g, controls jsonb, temperature_display, notes, created_at)

-- Bean catalog (global, community-owned)
beans (
  id uuid PK, created_by uuid FK,
  name, origin_country, origin_region, farm_producer, variety,
  altitude_masl int, processing text, supplier, lot_number, harvest_year int, notes,
  search_vector tsvector GENERATED ALWAYS AS (
    to_tsvector('english', coalesce(name,'') || ' ' || coalesce(origin_country,'') || ' ' ||
                           coalesce(origin_region,'') || ' ' || coalesce(variety,''))
  ) STORED,
  created_at
)
CREATE INDEX beans_search_idx ON beans USING gin(search_vector);

-- Roast sessions (user-scoped)
roasts (
  id uuid PK, user_id uuid FK, bean_id uuid FK, roaster_id uuid FK,
  date date, status text CHECK (status IN ('in_progress','roasted','cupped')),
  visibility text DEFAULT 'public' CHECK (visibility IN ('public','private')),
  previous_roast_id uuid REFERENCES roasts(id),
  variable_changed text, hypothesis text, intent text,
  -- settings
  initial_temp_f int, initial_airflow, initial_drum_speed, ambient_temp_f int, humidity_notes text,
  -- timeline
  yellowing_s int, first_crack_onset_s int, first_crack_end_s int,
  second_crack_onset_s int, drop_s int,
  total_time_s int GENERATED ALWAYS AS (drop_s) STORED,
  development_time_s int GENERATED ALWAYS AS (drop_s - first_crack_end_s) STORED,
  development_time_ratio numeric(5,3),  -- computed at API layer
  -- weights
  green_weight_g int, roasted_weight_g int, weight_loss_pct numeric(4,1),
  -- sensory
  aroma_drying text, aroma_maillard text, aroma_development text,
  color_at_yellowing text, color_at_drop text, crack_character text, observations text,
  -- post-roast
  target_roast_level text, roast_level_achieved text, aroma_cooled text,
  -- snapshots
  bean_snapshot jsonb,     -- frozen copy of bean at time of roast
  roaster_snapshot jsonb,  -- frozen copy of roaster at time of roast
  created_at timestamptz, updated_at timestamptz
)

-- Mid-roast adjustments (child of roast)
roast_adjustments (
  id uuid PK, roast_id uuid FK CASCADE,
  elapsed_s int, temp_f int, airflow, drum_speed, reason text, seq int
)

-- Cupping sessions (normalized from embedded array)
cuppings (
  id uuid PK, roast_id uuid FK CASCADE, user_id uuid FK,
  cupping_date date, rest_hours int, brew_method text,
  aroma_dry text, aroma_wet text,
  acidity_rating int, acidity_notes text,
  sweetness_rating int, sweetness_notes text,
  body_rating int, body_notes text,
  finish_length text CHECK (finish_length IN ('short','medium','long')),
  finish_quality text,
  flavor_descriptors text[], defect_flags text[],
  overall_rating int, tasting_notes_free text,
  hypothesis_result text, next_adjustment text,
  created_at timestamptz
)

-- Social graph
follows (
  follower_id uuid REFERENCES users(id),
  followee_id uuid REFERENCES users(id),
  created_at timestamptz,
  PRIMARY KEY(follower_id, followee_id)
)
CREATE INDEX follows_followee_idx ON follows(followee_id);
```

---

## Multi-User Data Isolation

| Entity | Scoped to User? | Rationale |
|--------|----------------|-----------|
| Roasters | YES | Equipment is personal |
| Roasts | YES | Sessions are personal |
| Cuppings | YES | Notes are personal |
| Beans | NO (global, `created_by` only) | A bean lot exists independently of who logs it |

Visibility enforcement: Application-layer `WHERE (visibility = 'public' OR user_id = $current_user_id)`. Simpler than PostgreSQL RLS and sufficient for a single-team codebase. Add RLS if an audit requires it.

---

## Build Order

```
1. DB schema + migrations
2. Auth (users, JWT issue/verify, session middleware)
3. Beans API (global, read-public) + Roasters API (user-scoped CRUD)
4. Roasts API (user-scoped, visibility filter) + Adjustments API
5. Cuppings API + Roast history chain (previous_roast_id traversal)
--- Shippable single-user logging app after step 5 ---
6. Social graph (follows, user profiles, feed query)
7. Bean search (full-text GIN index, shows public roasts per bean)
--- Community layer complete after step 7 ---
8. Live logging (WebSocket broadcast layer on top of existing Roasts API)
```

---

## JSON Prototype → Database Migration

1. Parse `beans.json` → INSERT into `beans` (assign UUIDs, set `created_by` to prototype user)
2. Parse `roasters.json` → INSERT into `roasters` (assign `user_id` to prototype user)
3. For each file in `data/roasts/`:
   - Insert into `roasts` (map all top-level fields, copy snapshots to JSONB columns unchanged)
   - Extract `adjustments[]` → INSERT into `roast_adjustments`
   - Extract `cuppings[]` → INSERT into `cuppings`
4. Resolve `previous_roast_id` string references (e.g. `"roast-20260115-001"`) to UUIDs using a temporary ID mapping table before finalizing FK constraints

The snapshot pattern means zero data loss. Frozen JSONB copies survive the migration intact.

---

## Anti-Patterns to Avoid

**Scoping beans to users** — Destroys the community bean search value proposition. Two users logging the same bean must reference the same catalog record.

**Fan-out feed at launch** — Adds Redis + background jobs before the social graph has real users. Pull query with index is correct until measured otherwise.

**WebSocket writes for live logging** — Roast events are durable state. REST writes + WebSocket broadcast is the correct separation.

**Real-time temperature curve streaming** — Requires hardware bridge (serial/USB). Out of scope per PROJECT.md.

---

## Scalability

| Concern | At 500 users | At 10K users | At 100K users |
|---------|-------------|--------------|---------------|
| Feed query | Pull join, fine | Pull join + index, fine | Consider fan-out table |
| Bean search | GIN index, fine | GIN index, fine | May need Typesense |
| Roast reads | Single DB, fine | Read replica | Read replica + cache |
| Live logging | Single server WS | Single server WS | Redis pubsub adapter |

Target realistic scale: 500–5,000 active users. Design for 10K, don't pre-optimize for 100K.

---

## Open Questions

- Authentication provider choice — belongs in STACK.md
- Whether Next.js app-router serves as full-stack or separate frontend + API
- Bean deduplication strategy — if two users add the same bean with slightly different spellings, no merge mechanism exists. Moderation problem, not architecture problem; not needed at launch.
