# Drumfire

## What This Is

Drumfire is an open-source coffee roast logging system for hobbyist roasters. It starts as a Claude-native prototype — no UI, no framework, just structured JSON files and Claude as the interface — so the right data model can be discovered through real use before anything is built on top of it. The long-term goal is a web app where roasters log their roasts, build public profiles, and discover how others approached the same bean.

## Core Value

Community discovery — a roaster can search any bean or origin and see exactly how other hobbyist roasters approached it: their settings, timeline, cupping notes, and what they'd change next.

## Requirements

### Validated

(None yet — ship to validate)

### Active

**Milestone 1 — Claude Prototype (current)**

- [ ] Roaster catalog: add and store roaster equipment with capabilities
- [ ] Bean catalog: add and store green coffee with origin, processing, variety
- [ ] Roast logging: create a roast record linking bean + roaster with settings
- [ ] Timeline tracking: log yellowing, first crack onset/end, second crack, drop as elapsed seconds
- [ ] Weight tracking: green weight in, roasted weight out, compute weight loss %
- [ ] Computed fields: development time, total time, DTR — auto-calculated when inputs exist
- [ ] Mid-roast adjustments: log temperature changes with elapsed time and reason
- [ ] Sensory notes: aroma, color, crack character during the roast
- [ ] Cupping sessions: guided tasting linked to a roast (aroma, acidity, body, finish, flavors, defects, rating)
- [ ] Hypothesis loop: record hypothesis before roasting, capture result after cupping
- [ ] Next adjustment: set a specific variable to change on the next roast of this bean
- [ ] Roast history: walk the full previous_roast_id chain for a bean, show timeline with ratings
- [ ] Planning: surface last adjustment, trend in ratings, recurring defects before a roast

**Milestone 2 — Web App (future)**

- [ ] User authentication: sign up, log in, stay logged in
- [ ] Public profiles: each roaster has a profile page with their roast history
- [ ] Roast visibility: public by default, option to make private
- [ ] Following: follow other roasters, see their activity in a feed
- [ ] Bean search: search by bean name or origin, see all public roast logs for that bean
- [ ] Live logging mode: timer running, log events as they happen in the browser

### Out of Scope

- Web or terminal UI in Milestone 1 — Claude is the interface while the data model is being validated
- User accounts in Milestone 1 — single-user local system by design
- Real-time roast curve graphing (Artisan-style) — out of scope for hobbyist target; timeline events are sufficient
- Marketplace or bean sourcing features — logging tool, not a shop

## Context

- **Current state**: Flat JSON files (`data/roasters.json`, `data/beans.json`, `data/roasts/roast-YYYYMMDD-NNN.json`). Claude reads and writes them directly.
- **Snapshot pattern**: Each roast file contains full frozen copies of the bean and roaster objects at time of roast. Historical records are self-contained and immune to catalog edits.
- **Cuppings embedded**: Cupping objects live in the `cuppings[]` array inside the roast file. No separate cupping entity.
- **Roast chain**: `previous_roast_id` links roasts of the same bean chronologically. Supports the hypothesis → adjust → re-roast iteration loop.
- **Bean/roaster independence**: Beans are global, not scoped to a roaster. A bag of green coffee exists independently of what machine it goes into.
- **Target user**: Hobbyist roasters using home machines (popcorn poppers, small drum roasters like the GeneCafe CB-101). Not professionals, not Artisan/Cropster users. Currently using spreadsheets or nothing.
- **Problem with existing tools**: Spreadsheets have no structure and no community. Pro tools (Artisan, Cropster) are overkill, expensive, and have no social layer.

## Constraints

- **Architecture (M1)**: No application framework — Claude is the only interface. Keeps iteration fast.
- **Data model (M1)**: Must be validated through real roasting sessions before the web app is built. The prototype exists to answer "what do I actually need to track?"
- **Open source**: MIT license, designed for community contribution once the web app phase begins.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| JSON flat files for storage | Discover the right data model through use, not upfront design | — Pending |
| Snapshot pattern (denormalized roast records) | Historical integrity — catalog edits shouldn't silently corrupt old roasts | — Pending |
| Cuppings embedded in roast files | Cuppings don't need to be queried independently; roast is the unit of record | — Pending |
| Beans global, not scoped to roaster | Green coffee is purchased once; same bag can be roasted on different machines | — Pending |
| Claude as v1 interface | Zero build time — get to real use immediately, validate before building | — Pending |
| Community discovery as core value | Differentiates from spreadsheets and pro tools; the social layer is the moat | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd:transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd:complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-04-26 after initialization*
