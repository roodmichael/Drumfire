# Pitfalls Research — Drumfire

**Domain:** Hobbyist coffee roast logging web app with social/community features
**Confidence:** HIGH for structural/architectural pitfalls; MEDIUM for community/UX-specific pitfalls

---

## Key Findings

- The snapshot pattern in the JSON prototype is correct and must survive migration — normalizing it away breaks historical record integrity.
- The community cold-start problem will kill the product if search is built before there's anything to find.
- Bean identity (who owns the canonical bean?) is the hardest data modeling decision and must be solved at schema design time.
- Social feature complexity has natural gravity — the surface must be frozen before any social code is written.
- Live logging does not require real-time infrastructure; a browser-local timer is the correct architecture.

---

## Critical Pitfalls

### 1. Data Model Lock-In from JSON Prototype (HIGH confidence)

**What goes wrong:** When migrating to Postgres, developers normalize away the snapshot pattern "for correctness." Historical records silently reference current catalog data instead of data at roast time. The `previous_roast_id` chain query isn't written until after launch, when adding the right index requires a table scan on live data.

**Warning signs:**
- "Let's just normalize the snapshots" in a PR review
- Bean search queries `bean_snapshot->>'origin_country'` instead of a canonical `beans` table
- Recursive CTE for roast chain written after schema is finalized

**Prevention:**
- Preserve `bean_snapshot` and `roaster_snapshot` as JSONB columns alongside FKs. Correct, not redundant.
- Design community search against canonical `beans` table, not snapshots.
- Write the recursive CTE before schema is finalized. Run `EXPLAIN ANALYZE`. Add the index.
- Test migration script against real M1 prototype data before any web app code.

**Phase:** M2 schema design — before any code is written.

---

### 2. Community Cold-Start (HIGH confidence)

**What goes wrong:** Community discovery requires other users' data. At launch with fewer than 50 public roasts, every search returns nothing. New users see an empty result and leave. The value proposition is invisible when it most needs to land.

**Warning signs:**
- Search is the first social feature built
- No seed data plan exists at launch
- Landing page promises community discovery with no evidence of community

**Prevention:**
- Import 20–50 real roast records from M1 prototype sessions as seed data before any public launch.
- Build logging first, community discovery second. App must be useful solo before it's useful communally.
- On empty bean search: "Be the first to log a roast of this bean" — turns a dead end into a call to action.
- Soft launch to a small known audience (r/roasting, home-barista.com) to seed 50–100 real records before broad availability.

**Phase:** M2 launch sequencing.

---

### 3. Social Feature Complexity Creep (HIGH confidence)

**What goes wrong:** Following ships. The feed needs likes. Likes need notifications. Notifications need email digests. Each sounds like a small addition; together they constitute a second product. Logging quality degrades as engineering attention splits.

**Warning signs:**
- "While we're adding following, let's add likes — it's just one more table"
- Notification system being designed before the feed works
- Any discussion of email scheduling in Milestone 2

**Prevention:**
- Freeze the M2 social surface explicitly: following + public profiles + bean search feed. Nothing else.
- Build social features as read-only views of logging data — no interaction layer in v1.
- Explicit out-of-scope for M2: likes, comments, reactions, direct messages, notifications, email digests.
- Test: "Does this make logging better, or does it make the social layer more social?" If the latter, defer.

**Phase:** M2 scoping — write the out-of-scope list before writing any social code.

---

### 4. Bean Identity Crisis (HIGH confidence)

**What goes wrong:** Each user creates their own bean entries. Hundreds of variants of the same bean with different names, spellings, and supplier references accumulate. Community search fragments and the value proposition fails.

**Warning signs:**
- `beans` table has a `user_id` column (user-scoped instead of global)
- No deduplication strategy exists
- Two users can independently create unlinked entries for the same bean

**Prevention:**
- Two-layer model: canonical `beans` table (shared, moderator-editable) + user-supplied metadata on the roast (lot number, supplier, harvest year, purchase date).
- Users search for and log against a canonical bean, rather than creating one from scratch.
- Lightweight moderation queue for new canonical bean suggestions.
- Personal catalog from M1 prototype maps to user-specific metadata on the roast — not a separate bean entity.

**Phase:** M2 schema design — hardest data modeling decision; must be resolved before any bean-related code.

---

### 5. Search That Doesn't Scale (HIGH confidence)

**What goes wrong:** Bean search implemented as SQL `ILIKE '%ethiopia%'`. Works with 50 records, degrades at 5,000. Also produces poor results because hobbyists enter origin data inconsistently ("Ethiopian Yirgacheffe" vs "Ethiopia, Yirgacheffe" vs "Yirgacheffe Ethiopia").

**Warning signs:**
- Bean search uses `LIKE` or `ILIKE`
- `origin_country` is a free-text field with no validation
- Full-text search added after schema is finalized

**Prevention:**
- PostgreSQL full-text search (`tsvector`/`tsquery`) from day one — handles stemming and ranking.
- `search_vector` generated column on `beans`, GIN index. Must be in the initial migration.
- `origin_country` as a controlled value (ISO country name or short enum); `origin_region` can be free text.

**Phase:** M2 schema design — `search_vector` GIN index in initial migration.

---

## Moderate Pitfalls

### 6. Over-Engineering Live Logging (HIGH confidence)

**What goes wrong:** Live logging triggers WebSocket thinking. Socket.io or Supabase Realtime gets added. Complexity exceeds the rest of the app combined. A roast generates 8–12 coarse events over 8–15 minutes. This is not a real-time data problem.

**Prevention:**
- Browser-local timer (`Date.now()`) + component state event log. Single save to server at drop time.
- Auto-save draft every 60 seconds as a safety net. Offer to restore on page reload.

**Phase:** M2 live logging design — decide local-first explicitly before backend work starts.

---

### 7. Auth Added After Data Model (HIGH confidence)

**What goes wrong:** "Build logging, add auth later." Logging is built against a hardcoded user. Auth added at week 4, requiring migration to add `user_id` to every table and rewrite every query.

**Prevention:**
- Auth is Milestone 2 task 1 — before any other table is created.
- Every table except canonical `beans` gets a `user_id` from day one.

**Phase:** M2 task sequencing.

---

### 8. Public Profiles Without Per-Roast Privacy Toggle (MEDIUM confidence)

**What goes wrong:** "Public by default" ships without per-roast opt-out. Users' experimental failures and process notes are visible to their bean suppliers. Trust damage spreads in a small hobbyist community.

**Prevention:**
- `visibility` column on roasts table (`public` | `private`) in the initial migration.
- One-tap toggle in the logging UI.
- Private roasts never appear in bean search, feeds, or profiles.

**Phase:** M2 schema design — `visibility` in initial migration.

---

## Minor Pitfalls

| # | Pitfall | Prevention |
|---|---------|-----------|
| 9 | Hypothesis/next_adjustment visible in public views — reads as unpolished process notes | Show owner-only; public views show cupping notes and settings, not working notes |
| 10 | DTR displayed without reference range — new users see it as noise | Always show DTR with inline reference range from the cheatsheet |
| 11 | Weight loss % as roast level proxy — users optimize the number, not the cup | Show alongside `roast_level_achieved`; never as substitute for human judgment |

---

## Phase-Specific Warnings

| Phase | Likely Pitfall | Mitigation |
|-------|----------------|------------|
| M2: Schema design | Bean identity fragmentation | Two-layer model before any code |
| M2: Schema design | Missing `visibility` column | Add to initial migration |
| M2: Schema design | Missing full-text search index | `search_vector` GIN index in initial migration |
| M2: First tasks | Auth after data model | Auth is task 1 |
| M2: Migration | Snapshot pattern normalized away | Preserve JSONB snapshots alongside FKs |
| M2: Live logging | WebSocket over-engineering | Local-first timer, single save at drop |
| M2: Launch | Empty search at launch | Seed 20–50 real records before go-live |
| M2: Scope | Social feature creep | Freeze social surface; explicit out-of-scope list |
