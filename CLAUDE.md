# Drumfire

A personal coffee roast logging system. The user talks naturally; Claude reads and writes structured JSON files. No application framework — Claude is the interface.

**Goal:** Discover the right data model through real use, then migrate to a web or terminal app later.

---

## Slash Commands

Three focused modes — use these instead of general conversation when doing a full session:

- `/plan` — pre-roast planning: reviews bean history, surfaces last adjustment, sets hypothesis
- `/roast` — active roasting: real-time logging, phase guidance, live diagnostics
- `/cup` — cupping session: guided tasting, connects cup to roast, sets next adjustment

## Reference Files

- `cheatsheet.md` — roasting reference card (phases, roast levels, weight loss, defect table)
- `troubleshooting.md` — deep diagnostic guide: CB-101 behavior, phase-by-phase problems, defect profiles, cup→roast and roast→cup cross-references

---

## File Structure

```
data/
  roasters.json       # roasting equipment catalog
  beans.json          # green bean catalog
  roasts/             # one JSON file per roast
    roast-YYYYMMDD-NNN.json
cheatsheet.md         # roasting reference card (display on request)
CLAUDE.md             # this file
```

---

## Interaction Patterns

### "add roaster" / "new roaster" / no roaster exists yet
Collect and save to `roasters.json`:
- name, manufacturer, model
- type: `drum` | `air` | `stovetop` | `popcorn-popper` | `other`
- capacity_min_g, capacity_max_g
- controls: which of temperature / airflow / drum_speed / timer the machine has
- temperature_display: what the readout represents (e.g. "air temperature, not bean temp")
- notes

### "new roast" / "start roasting" / "I'm about to roast"
1. If one roaster exists, use it. If multiple, ask which.
2. Check `beans.json`. If bean exists, pre-populate snapshot. If new, collect bean fields and add to catalog.
3. If bean has previous roasts, find the most recent one and surface its `next_adjustment`. Ask if the user is applying it (sets `variable_changed` and `hypothesis`).
4. Only ask for setting fields the roaster's `controls` object supports.
5. Create `data/roasts/roast-YYYYMMDD-NNN.json` with status `in_progress`.
6. Offer to display the cheatsheet.

### Logging observations mid-roast or retrospectively
Parse natural language into fields. Accept either:
- **Clock times** (e.g. "yellow at 7:32") — store as elapsed seconds from charge if charge time is known, otherwise store the clock time and calculate later
- **Elapsed times** (e.g. "yellow at 7:30 into the roast") — store directly as seconds

Compute derived fields as soon as the required inputs exist:
- `development_time_s` = `drop_s` − `first_crack_end_s`
- `total_time_s` = `drop_s`
- `development_time_ratio` = `development_time_s` / `total_time_s` (round to 3 decimal places)

### "dropped" / "roasted weight is Xg"
- Record drop time and/or roasted weight
- Compute `weight_loss_pct` = (`green_weight_g` − `roasted_weight_g`) / `green_weight_g` × 100 (round to 1 decimal)
- Set status to `roasted`

### "cup the [bean/date] roast" / "add tasting notes" / "cupping"
- Find roast by bean name, date, partial id, or "most recent"
- Walk through cupping fields (aroma, acidity, sweetness, body, finish, flavors, defects, overall, hypothesis result, next adjustment)
- Append to the `cuppings` array in the roast file
- If this roast has a `hypothesis`, show it alongside the new `hypothesis_result` for comparison
- Set status to `cupped`

### "last time I roasted X" / "history for X" / "show my roasts"
- Read relevant roast files
- For a specific bean: walk the `previous_roast_id` chain and present as a timeline with ratings and key variables
- For all roasts: summarize sorted by date

### "plan my next roast of X" / "what should I adjust"
- Walk the full `previous_roast_id` chain for that bean
- Surface: last `next_adjustment`, trend in `overall_rating`, recurring `defect_flags`
- Suggest one specific variable to change and the predicted cup effect

### "cheatsheet" / "reference card" / "what should I look for"
- Display the contents of `cheatsheet.md`

---

## Roasters Schema (`roasters.json`)

```json
[
  {
    "id": "roaster-001",
    "name": "",
    "manufacturer": "",
    "model": "",
    "type": "",
    "capacity_min_g": null,
    "capacity_max_g": null,
    "controls": {
      "temperature": true,
      "airflow": false,
      "drum_speed": false,
      "timer": true
    },
    "temperature_display": "",
    "notes": ""
  }
]
```

---

## Beans Schema (`beans.json`)

```json
[
  {
    "id": "bean-001",
    "name": "",
    "origin_country": "",
    "origin_region": "",
    "farm_producer": "",
    "variety": "",
    "altitude_masl": null,
    "processing": "",
    "supplier": "",
    "lot_number": "",
    "harvest_year": null,
    "purchase_date": "",
    "notes": ""
  }
]
```

`processing` enum: `washed` | `natural` | `honey` | `anaerobic` | `wet-hulled` | `other`

---

## Roast Schema (`data/roasts/roast-YYYYMMDD-NNN.json`)

```json
{
  "id": "roast-20260115-001",
  "date": "2026-01-15",
  "status": "in_progress",

  "bean_id": "bean-001",
  "bean_snapshot": {},

  "roaster_id": "roaster-001",
  "roaster_snapshot": {},

  "intent": "",
  "previous_roast_id": null,
  "variable_changed": null,
  "hypothesis": null,

  "settings": {
    "initial_temp_f": null,
    "initial_airflow": null,
    "initial_drum_speed": null,
    "ambient_temp_f": null,
    "humidity_notes": ""
  },

  "adjustments": [
    { "elapsed_s": null, "temp_f": null, "airflow": null, "drum_speed": null, "reason": "" }
  ],

  "green_weight_g": null,

  "timeline": {
    "yellowing_s": null,
    "first_crack_onset_s": null,
    "first_crack_end_s": null,
    "second_crack_onset_s": null,
    "drop_s": null,
    "total_time_s": null,
    "development_time_s": null,
    "development_time_ratio": null
  },

  "sensory": {
    "aroma_drying": "",
    "aroma_maillard": "",
    "aroma_development": "",
    "color_at_yellowing": "",
    "color_at_drop": "",
    "crack_character": "",
    "observations": ""
  },

  "post_roast": {
    "roasted_weight_g": null,
    "weight_loss_pct": null,
    "target_roast_level": "",
    "roast_level_achieved": "",
    "aroma_cooled": ""
  },

  "cuppings": []
}
```

**Cupping object** (appended to `cuppings` array):
```json
{
  "cupping_date": "",
  "rest_hours": null,
  "brew_method": "",
  "aroma_dry": "",
  "aroma_wet": "",
  "acidity": { "rating": null, "notes": "" },
  "sweetness": { "rating": null, "notes": "" },
  "body": { "rating": null, "notes": "" },
  "finish_length": "",
  "finish_quality": "",
  "flavor_descriptors": [],
  "defect_flags": [],
  "overall_rating": null,
  "tasting_notes_free": "",
  "hypothesis_result": "",
  "next_adjustment": ""
}
```

**Enums:**
- `status`: `in_progress` | `roasted` | `cupped`
- `roast_level_achieved`: `light` | `medium-light` | `medium` | `medium-dark` | `dark`
- `finish_length`: `short` | `medium` | `long`
- `defect_flags`: `underdeveloped` | `baked` | `scorched` | `overdeveloped` | `grassy` | `papery` | `astringent`

**Computed fields** (always calculate and write, never leave null if inputs are available):
- `development_time_s` = `drop_s` − `first_crack_end_s`
- `total_time_s` = `drop_s`
- `development_time_ratio` = `development_time_s` / `total_time_s`
- `weight_loss_pct` = (`green_weight_g` − `roasted_weight_g`) / `green_weight_g` × 100

**ID format:** `roast-YYYYMMDD-NNN` where NNN is zero-padded sequence for that date (001, 002, …). Check existing files in `data/roasts/` to determine the next sequence number.

**Snapshots:** When creating a roast, copy the full bean and roaster objects from their catalogs into `bean_snapshot` and `roaster_snapshot`. This preserves the record even if catalog entries are later edited.

**Adjustment loop:** `previous_roast_id` links roasts of the same bean into a chain. When starting a new roast of a known bean, look up all roasts with that `bean_id`, find the most recent by date, and use its `id` as `previous_roast_id`.

**Empty adjustments:** If no mid-roast temperature changes occurred, set `adjustments` to `[]`.
