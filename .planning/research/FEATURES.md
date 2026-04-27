# Features Research — Drumfire

**Domain:** Hobbyist Coffee Roast Logging SaaS
**Confidence:** MEDIUM-HIGH overall

---

## Existing Tool Survey

**Artisan** (free, open-source desktop) — HIGH confidence
Real-time roast curves via hardware probes (TC4, Phidgets), RoR calculation, event markers. Requires USB probe hardware most home machines (popcorn poppers, GeneCafe) cannot accommodate. Zero social layer. No bean catalog, no cupping notes, no community. Every roaster is an island. Right answer for the subset who can wire probes in; no answer for "what did others do with this bean?"

**Cropster** (~$100-200+/month SaaS) — HIGH confidence
Commercial roastery software. Production logging, green coffee inventory, blend management, QC workflows, SCA cupping forms. Price eliminates hobbyists at the door. No social/community features. Defines the professional ceiling; Drumfire's community layer is explicitly what Cropster doesn't do.

**RoastLog** (web-based freemium) — MEDIUM confidence
Closest spiritual predecessor. Web-based, green coffee catalog, roast history, basic charts. More approachable than Cropster but still oriented toward small roasteries. Social features exist but are not community-forward — profiles without a discovery layer. Active development slowed circa 2023-2024. Its stagnation is an opening.

**Typica** (desktop, open source) — HIGH confidence
Production record-keeping for small roasteries. Green coffee purchasing, customer management, batch logs. Not designed for iterative hobbyist experimentation. No cupping workflow. Not relevant to Drumfire's target.

**Roast.World** — MEDIUM confidence
Proves the community sharing model works. Roasters browse public roast profiles by bean — exactly Drumfire's community value prop. Heavily skewed toward Ikawa sample roasters. Cupping and hypothesis loop are absent. Confirms demand exists for community bean discovery.

---

## Table Stakes

Features users expect. Missing = product feels incomplete or untrustworthy.

| Feature | Why Expected | Complexity |
|---------|--------------|------------|
| Roast log creation | Core function | Low |
| Bean catalog | Same bean roasted many times; must track it | Low |
| Timeline event logging | First crack, drop time are the language of roasting | Low |
| Weight tracking (green/roasted) | Weight loss % is a standard quality signal | Low |
| Roast history per bean | "What did I do last time?" | Low |
| Cupping notes entry | Feedback loop closes through tasting | Medium |
| Defect flagging | Named flags beat free text for troubleshooting | Low |
| Overall rating | Single score for trend tracking | Low |
| Next adjustment field | Captures the hypothesis→adjust loop | Low |
| Public/private toggle per roast | Some experiments users don't want public | Low |
| User authentication | Required for multi-user | Medium |
| User profile page | Other roasters need a destination to visit | Medium |
| Mobile-friendly UI | Roasting happens in the kitchen, not at a desk | Medium |

---

## Differentiators

| Feature | Value Proposition | Complexity |
|---------|-------------------|------------|
| Bean search → community roasts | Search any bean/origin, see every public log — the core moat | High |
| Hypothesis loop (structured) | Explicit before/after scientific thinking; no other hobbyist tool does this | Low |
| Roast chain visualization | Full iteration history of a bean as a timeline with ratings | Medium |
| Social follow + feed | See activity from roasters you follow | High |
| "What others changed" aggregation | Surface patterns in next_adjustment across all public roasts of a bean | High |
| Roaster equipment filtering | "How did other GeneCafe CB-101 users approach this bean?" | Medium |
| Defect pattern surfacing | "You always get baked notes from this origin" — within user history | Medium |
| Planning mode | Pre-roast brief: last adjustment, rating trend, recurring defects | Low |
| Live logging mode | Timer running in browser, log events in real time | Medium |
| Cupping comparison | Side-by-side view of two cuppings before/after an adjustment | Medium |
| Bean discovery / origin browsing | Browse public roasts by country or region | Medium |
| Export (CSV/JSON) | Data portability builds trust | Low |

---

## Anti-Features

Features to deliberately NOT build.

| Anti-Feature | Why Avoid |
|--------------|-----------|
| Real-time roast curve graphing (Artisan-style) | Requires hardware probes most home roasters can't do; Artisan does this free |
| Hardware device integration | Platform-specific installs, hardware SKU explosion — Artisan's entire complexity budget |
| Green coffee inventory / bag tracking | Cropster/Typica do this for businesses; hobbyists don't need stock levels |
| Blend tracking | Roasteries blend; hobbyists roast single origins |
| Customer / sales management | Hobbyists do not sell commercially |
| SCA scoring form (10-attribute) | Q-grader protocol is for professionals; wrong register entirely |
| Subscription tiers with paywalled logging | Paywalling core function creates churn before value is delivered |
| Marketplace / bean sourcing | Separate business; conflicts with supplier neutrality |
| Native mobile app (iOS/Android) | PWA-quality responsive web is sufficient for v1 |
| AI-generated roast recommendations | Requires data density that belongs in v3 at earliest; wrong often enough to damage trust |
| Engagement metrics / likes | Gamification is a distraction for a logging tool |

---

## Feature Dependencies

```
User auth → Public profiles → Roast visibility control → Bean community search → Social follow + feed
Bean catalog → Roast log creation → Timeline logging → Computed fields (DTR, dev time, weight loss %)
Roast log creation → Cupping notes → Hypothesis result → Next adjustment
Roast history chain → Planning mode
Roast history chain → Defect pattern surfacing
Bean community search → "What others changed" aggregation (needs data density)
Live logging mode → Mobile-friendly UI (prerequisite)
```

---

## MVP Recommendation (Milestone 2)

**Ship at launch:**
1. User auth (email + Google OAuth)
2. Bean catalog + roast log creation (port M1 data model)
3. Timeline + weight logging with computed fields
4. Cupping notes with defect flags and overall rating
5. Hypothesis loop (hypothesis and hypothesis_result displayed together — the feature that makes Drumfire feel different)
6. Roast history per bean (chain view with rating trend)
7. Public profile page
8. Bean search showing all public roasts — even as a simple list; this is the community hook that must exist at launch

**Defer:**
- Social follow + feed: needs user volume first
- "What others changed" aggregation: needs data density (year 2+)
- Live logging mode: important but not launch-blocking; web form works for post-roast entry
- Cupping comparison view: data supports it from day one; UI can come post-launch

---

## Hobbyist Segment Notes

- **Machine capability**: Popcorn poppers, stovetop, GeneCafe CB-101, Behmor 1600 — no probe ports. Hardware integration impossible and unnecessary.
- **Volume**: 1–4 roasts per week, 50–250g batches.
- **Goal**: Personal enjoyment and skill development. Not commercial QC.
- **Current baseline**: Google Sheets, paper notebooks, or nothing.
- **Community need**: r/roasting posts ("here's what I did with this Ethiopia, here's what I'd change") are manual versions of exactly what Drumfire makes structured and searchable.
- **Device context**: Roasting happens in the garage or kitchen. Phone is the available device. Mobile UI is not optional.

---

## Open Questions

- Does any post-August 2025 entrant now offer the hypothesis loop or structured cupping comparison?
- What is the data density threshold for "What others changed" aggregation to be useful?
- Do hobbyists prefer 1–10 or 100-point cupping rating scales?
