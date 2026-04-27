# Stack Research — Drumfire

**Domain:** Community Coffee Roast Logging SaaS (Milestone 2)
**Confidence:** MEDIUM-HIGH (Next.js 16.2 and Zod verified via official docs; other versions from training data through Aug 2025)

---

## Key Findings

- **Next.js 16.2** (App Router + Server Components) is the right framework — public profile/bean search pages need server rendering; logged-in logging pages need interactivity. Both in one project.
- **Better Auth** is in the official Next.js authentication guide — self-hosted, no per-MAU pricing, works with App Router. Right choice for an open-source community app.
- **Drizzle ORM + Neon (serverless Postgres)** is the 2025 standard for Next.js on Vercel. Drizzle's SQL-first approach is important because the schema is migrating from a validated JSON prototype — full visibility into every query.
- **No WebSocket layer needed for live logging.** A browser-side timer with `useOptimistic` + Server Actions is sufficient. No concurrent viewers during a roast.
- **Postgres full-text search** (GIN index on `tsvector` column) is sufficient at Milestone 2 scale. Typesense is the upgrade path if fuzzy matching becomes a pain point.

---

## Core Framework

**Next.js 16.2, App Router** — CONFIDENCE: HIGH (verified)

App Router is correct because:
- Public pages (profiles, bean search, individual roast pages) → Server Components → zero client JS → fast and SEO-friendly
- Logged-in pages (new roast, cupping entry, live logging) → Client Components where needed
- Server Actions replace a separate REST API for mutations — fewer moving parts

Turbopack is stable and default in 16.2 (~400% faster dev startup).

**TypeScript 5.x** — CONFIDENCE: HIGH

The data model has strict enums (`washed | natural | honey | ...`, `underdeveloped | baked | scorched | ...`) that must be enforced at compile time when migrating to Postgres. Type errors prevent data corruption.

**React 19** — CONFIDENCE: HIGH (bundled with Next.js 16.x)

`useActionState` (auth forms) and `useOptimistic` (live logging feel) are both React 19 features that are used in the official Next.js examples.

---

## Database

**PostgreSQL 16 via Neon (serverless)** — CONFIDENCE: MEDIUM

Neon is the right hosting choice for Next.js on Vercel:
1. Serverless connection pooling — Next.js serverless functions exhaust traditional Postgres connection limits; Neon handles this automatically
2. Database branching — each PR gets its own Neon branch for integration testing
3. Free tier adequate for early community growth

The data model is highly relational (beans global, roasters user-scoped, roasts join both with timeline + adjustments + cuppings). This is a strong Postgres fit.

**PlanetScale eliminated** — dropped free tier 2024, uses MySQL not Postgres.

---

## ORM

**Drizzle ORM 0.30+** — CONFIDENCE: MEDIUM

Drizzle over Prisma because:
- SQL-first query builder — full visibility into every operation when migrating a validated JSON prototype to Postgres
- TypeScript schema = same type system that enforces roast field validity enforces DB column access
- `drizzle-kit` migrations are simpler than Prisma's for complex schema evolutions
- No N+1 hiding
- Better JSONB support — important for the snapshot pattern (frozen bean/roaster copies in roast records)

---

## Authentication

**Better Auth 1.x** — CONFIDENCE: MEDIUM (listed in official Next.js auth guide, verified April 2026)

- Self-hosted — no vendor lock-in
- Official Next.js App Router adapter
- Email/password + OAuth (Google, GitHub) out of the box
- No per-MAU pricing — community growth not penalized

**Clerk rejected** — per-MAU pricing ($0.02/MAU beyond 10k users) penalizes community growth.
**Auth.js (NextAuth v5) rejected** — RC-quality stability through mid-2025.
**Supabase Auth rejected** — creates pressure to use Supabase's whole stack, displacing Neon.

---

## Search

**PostgreSQL full-text search (built-in)** — CONFIDENCE: HIGH

Bean catalog query is structured: origin country, region, variety, name. A `tsvector` column with a GIN index handles autocomplete and fuzzy search without an external service.

```sql
beans.search_vector tsvector GENERATED ALWAYS AS (
  to_tsvector('english', coalesce(name,'') || ' ' || coalesce(origin_country,'') || ' ' ||
                         coalesce(origin_region,'') || ' ' || coalesce(variety,''))
) STORED

CREATE INDEX beans_search_idx ON beans USING gin(search_vector);
```

**Scale trigger:** If queries regularly exceed 200ms at ~50k bean entries, add **Typesense** (self-hostable). Do not add Algolia — per-operation pricing is unnecessary at this scale.

---

## Real-time / Live Logging

**React state + `useOptimistic` + Server Actions** — CONFIDENCE: HIGH

The requirement: browser-side timer, user taps to log events (yellowing, first crack, drop). Single-user session — no concurrent viewers.

```
useState(elapsedSeconds) + useEffect(interval, 1s)
→ user taps event
→ useOptimistic shows event immediately
→ Server Action writes to Postgres
→ component revalidates with actual DB state
```

No WebSocket or SSE infrastructure needed. If shared live viewing ever becomes a requirement, evaluate Supabase Realtime or Ably at that point.

---

## Styling

**Tailwind CSS 4.x** — CONFIDENCE: MEDIUM

v4 ships with zero config file — the CSS itself is the config. Utility classes compose well for diverse UI states: 5 roast level indicators, defect badge variants, timeline phase colors.

**shadcn/ui (latest)** — CONFIDENCE: MEDIUM

shadcn/ui copies component source directly into the project. Important because Drumfire's forms are non-standard: nested `adjustments[]` array, conditional fields based on roaster capabilities (airflow only if the roaster supports it). Owning the component code means customizing without fighting a library's API.

---

## Infrastructure

**Vercel** — CONFIDENCE: HIGH

Zero config for Next.js. Built-in preview deployments. CDN edge caching for public profile and bean search pages. Free tier sufficient for early growth.

**Vercel Blob** for object storage — CONFIDENCE: MEDIUM

Profile photos only. Cloudflare R2 is the cost-efficient alternative (no egress fees) if costs become material at scale.

---

## Validation

**Zod 3.x** — CONFIDENCE: HIGH (used in official Next.js auth guide, verified April 2026)

Zod schemas defined once, shared between client-side form validation and server-side Server Action validation. Standard Next.js pattern.

---

## Supporting Libraries

| Library | Purpose | Notes |
|---------|---------|-------|
| `react-hook-form 7.x` | Form handling | `useFieldArray` handles nested `adjustments[]` array; uncontrolled forms do not |
| `date-fns 3.x` | Date utilities | Roast date-keying, rest time calculations |
| `recharts 2.x` | Timeline visualization | Horizontal event timeline (yellowing → FC → drop) — defer until visualization phase |
| `jose 5.x` | JWT handling | Edge Runtime compatible; used in official Next.js session examples |

---

## Alternatives Rejected

| Category | Rejected | Reason |
|----------|---------|--------|
| Framework | Remix | Next.js App Router has better performance for read-heavy public pages |
| Framework | SvelteKit | Thinner TypeScript ecosystem |
| ORM | Prisma | Black-box queries, weaker JSONB support, N+1 hiding |
| Auth | Clerk | Per-MAU pricing penalizes community growth |
| Auth | Auth.js v5 | RC-quality through mid-2025 |
| Database | PlanetScale | Dropped free tier 2024, uses MySQL |
| Database | Supabase (full) | Bundles features overlapping with chosen stack |
| Search | Algolia | Per-operation pricing unnecessary at this scale |
| Real-time | Socket.io / Supabase Realtime | Over-engineered for single-user live logging |

---

## Installation Skeleton

```bash
npx create-next-app@latest drumfire-web --typescript --tailwind --app --src-dir

npm install drizzle-orm @neondatabase/serverless
npm install -D drizzle-kit
npm install better-auth
npm install zod
npm install react-hook-form @hookform/resolvers
npm install date-fns
npm install recharts  # defer until visualization phase
```

---

## Roadmap Implications

1. **Schema migration phase first** — Translate M1 JSON model into Drizzle table definitions. Key decision: keep JSONB snapshots or normalize into versioned records.
2. **Auth before social features** — Public profiles, following, feed all depend on user identity.
3. **Bean search is Postgres FTS from day one** — GIN index is part of the schema migration, no dedicated search phase.
4. **Live logging is UI-only** — No backend changes needed; event logging already exists in the data model.
5. **Social features are the most complex phase** — Feed query optimization and fan-out strategy need deeper research when that phase arrives.

---

## Open Questions

- **Snapshot pattern decision**: Keep JSONB snapshots in roast table (mirrors current file structure) or normalize into versioned bean/roaster records? Most consequential schema decision.
- **Better Auth current version**: Verify `1.x` stability and App Router adapter quality at time of M2 build.
- **Neon pricing**: Confirm free tier limits and connection pooling behavior with Next.js serverless functions.
- **Feed fanout threshold**: At what follower count does a pull query become slow? Research before social phase.
