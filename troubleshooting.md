# Roast Troubleshooting Guide

## How to Use This Guide

Use this guide in three situations:
1. **During a roast** — something looks or smells off; you want real-time guidance
2. **After a roast, before cupping** — the roast log has numbers you want to interpret
3. **During cupping** — you're tasting something wrong and want to trace it back to the roast

Always start from symptoms and work backward to cause. Don't assume a defect until you've tasted it — the log can suggest risk, but only the cup confirms it.

---

## Understanding the Gene Cafe CB-101

Knowing how this machine works prevents misreading its behavior as a defect.

**Temperature sensor reads air, not beans.** The dial and display show the temperature of the forced air moving through the drum, not the bean surface or bean core temperature. Actual bean temperature lags behind the displayed temperature, especially in the early phases. A displayed 400°F does not mean the beans are at 400°F.

**Temperature changes have significant lag.** When you increase or decrease the dial, the beans don't feel the change immediately. Allow 60–90 seconds for a temperature adjustment to fully manifest in bean behavior. Avoid chasing numbers with rapid adjustments — you'll overshoot.

**The drum provides even heat distribution.** Continuous tumbling means relatively even roast development across the batch, but this only holds within the recommended batch size range. Overloading prevents free tumbling and creates uneven roasts.

**The CB-101 loses momentum easily.** If you reduce temperature too aggressively — or at the wrong time — the roast can stall in the Maillard phase. Once stalled, the roast rarely fully recovers. The result is a baked profile even if your development time looks normal.

**Forced air means relatively fast roast times.** Expect total roast times of 12–16 minutes for 200–250g green. Significantly longer suggests you're running too cool and risking a bake. Significantly shorter suggests you're running too hot and risking a scorch.

**The cooling cycle uses the same drum.** Once you stop roasting and the cooling cycle begins, beans continue to absorb heat from the hot drum briefly before cooling dominates. Factor this in: if you're targeting a light roast, drop a few seconds earlier than you think.

**Between-batch temperature matters.** The CB-101 retains heat between roasts. If you roast two batches in succession, the second batch will encounter a hotter initial environment, affecting the drying phase rate. Allow the machine to cool to your target charge temperature before the second batch, or adjust your initial temperature setting down.

---

## CB-101 Typical Benchmarks (200–250g green)

These are reference ranges, not rigid targets. Your specific machine, altitude, ambient temperature, and bean density will shift them. Track your own benchmarks over time — your actuals matter more than these generalized numbers.

| Milestone | Typical Range | Notes |
|---|---|---|
| Total roast time | 12–16 min | Outside this range is a warning sign |
| Time to yellowing | 5–8 min | ~35–50% of total time |
| Time to first crack onset | 10–13 min | |
| Development time (light) | 45–75 sec | FC end to drop |
| Development time (medium) | 90–150 sec | FC end to drop |
| DTR (development time ratio) | 0.15–0.25 | Development ÷ total |
| Weight loss — light | 12–14% | |
| Weight loss — medium | 15–17% | |
| Weight loss — medium-dark | 17–20% | |

---

## Reading a Roast Log for Defects

Before cupping, you can read the numbers in a roast log to identify likely problems. This is not a substitute for tasting — it's a pre-screen that tells you what to look for.

### Step 1: Check total roast time
- **< 11 min:** Too fast. High scorch/tip risk. Look for harsh bitterness, rubber notes in the cup.
- **12–16 min:** Normal range.
- **> 17 min:** Too slow. High bake risk. Look for flat, papery, dull cup.

### Step 2: Calculate phase ratios
Using the timeline fields:
- **Drying ratio** = `yellowing_s` ÷ `total_time_s`
  - Target: 0.30–0.45
  - > 0.50: Drying too slow — elevated bake risk
  - < 0.25: Drying too fast — elevated scorch risk
- **Maillard ratio** = (`first_crack_onset_s` − `yellowing_s`) ÷ `total_time_s`
  - Target: 0.35–0.45
  - < 0.30: Maillard abbreviated — expect thin body and sharp acidity
- **DTR** = `development_time_s` ÷ `total_time_s`
  - Target: 0.15–0.25
  - < 0.15: Underdevelopment risk
  - > 0.25: Overdevelopment risk

### Step 3: Check weight loss %
- Significantly below the expected range for the target roast level: likely underdeveloped or pulled too early
- Significantly above: likely overdeveloped

### Step 4: Review sensory notes during roast
- Did aroma stall at any point (e.g., stuck at "bready" for a long time)? → Maillard stall, bake risk
- Did the development aroma never fully develop (stayed grassy/cereal)? → Underdevelopment risk
- Were there temperature adjustments? → Consider their timing and whether they disrupted momentum

### Step 5: Review temperature adjustments
- A temperature reduction during the Maillard phase that wasn't compensated is the most common CB-101 bake cause
- A temperature increase during development that extended past the planned drop point creates overdevelopment
- Multiple adjustments in quick succession suggests chasing — a sign the roast may be inconsistent

---

## Phase-by-Phase Diagnostics

### Drying Phase Problems

**Signs during roast:**
- Beans staying green/pale for longer than expected (7+ min without yellowing)
- No aroma development — the air smells like nothing or cool grain
- Temperature display not climbing despite time passing

**What it means:**
The drying phase is taking too long. The roast is at risk of stalling. The beans are spending too long in a low-energy state, which degrades volatile compounds before they can be properly developed later.

**CB-101 cause:**
Initial temperature set too low, or machine not fully pre-heated to charge temperature before loading beans.

**Intervention during the roast:**
Increase temperature by 10–15°F. Don't make a large jump — you want to restore momentum, not overshoot into scorching. Watch for the aroma to begin progressing (grassy → hay).

**If you didn't catch it:**
The roast may be baked regardless of what you do later. Document the timeline accurately and cup with low expectations. The fix is in the next roast, not this one.

---

**Signs during roast:**
- Beans turn yellow very quickly (< 4 min) with little aroma buildup
- Harsh, slightly acrid smell in early phases

**What it means:**
Drying too fast. The bean surface is being scorched before the interior moisture has a chance to migrate out properly.

**CB-101 cause:**
Initial temperature too high, or charging into a machine that was still hot from a previous roast.

**Intervention during the roast:**
Reduce temperature immediately. You can't un-scorch, but you can prevent it from getting worse.

---

### Maillard / Browning Phase Problems

**Signs during roast:**
- Color progression slows or stalls (stuck at tan/cinnamon for a long time)
- Aroma stalls at "bready" or "toasty" and doesn't progress toward caramel/chocolate
- Expected first crack time passes with no crack

**What it means:**
The roast has stalled. The Maillard reaction is running slowly or has effectively stopped. This is the most common cause of baked coffee on the CB-101 and one of the hardest defects to taste (it's defined by the absence of good flavors, not the presence of bad ones).

**CB-101 cause:**
Temperature was reduced too aggressively during this phase — often in response to a misread or an attempt to "slow down" before first crack. Once momentum is lost in this phase, it's very difficult to recover.

**Intervention during the roast:**
Increase temperature by 10–15°F immediately. Note the time of the stall in your observations. Be aware this roast may be baked regardless — document honestly.

**If you didn't catch it:**
When you cup: expect flat, papery, dull. Low sweetness, muted acidity, nothing interesting. The cup won't be offensive, just empty. Fix for next roast: don't reduce temperature during the Maillard phase unless you have a specific, intentional reason.

---

### Development Phase Problems

**Signs during roast:**
- First crack is very quiet, sparse, weak — just a few isolated pops rather than a rolling crack
- First crack ends very quickly (< 30 seconds)
- Crack never fully "rolls"

**What it means:**
The crack character reflects the energy state of the beans. A weak crack can indicate the beans entered development phase without sufficient energy built up in the Maillard phase. This is a symptom, not a cause — the problem likely started earlier.

---

**Signs during roast:**
- First crack is very loud, aggressive, almost explosive
- Crack rushes through quickly and second crack follows soon after

**What it means:**
The beans entered first crack with too much energy (too hot, too fast). The development phase will be compressed and harder to control. Drop precision matters more here — you have less time to act.

**CB-101 cause:**
Temperature too high entering development. The forced-air design can drive heat aggressively once the exothermic first crack begins.

**Intervention:**
Drop temperature slightly at FC onset to slow the pace. Have your timing ready — you'll need to drop at a precise moment.

---

## Defect Profiles

### Baked

**Cup symptoms:** Flat, papery, cardboard-like, muted, dull. Sweetness absent or very low. Acidity absent or very low — but not in a pleasant way, just empty. The cup lacks dimension. Nothing distinctly wrong, just nothing right.

**Roast log indicators:**
- Total roast time > 16–17 min
- Drying ratio > 0.50 (took too long to yellow)
- Observations noting stalled aroma progression
- Temperature reduction during Maillard phase

**CB-101 specific causes:**
- Starting temperature too low
- Temperature reduced too early or too aggressively, usually during browning
- Second batch without allowing machine to return to charge temperature
- Roasting on a very cold day without accounting for ambient temperature

**Root mechanism:**
The Maillard reaction is rate-sensitive. When heat input drops below what the endothermic bean can sustain, the reaction slows and the volatile aromatic compounds that would become sweetness and complexity degrade or fail to form. The cup tastes like the coffee was never properly developed — because it wasn't.

**Fix:**
Maintain temperature momentum through the entire Maillard phase. On the CB-101, this typically means not reducing temperature until you're at or near first crack onset. If you do reduce temperature, do it minimally (5–10°F) and watch for aroma stall.

---

### Underdeveloped

**Cup symptoms:** Sour, grassy, harsh, sharp, vegetal. Aggressive brightness that reads as unpleasant rather than lively. Cereal-like or raw grain notes. Dry finish. Often confused with "too light a roast" but the sourness is harsher and more aggressive than the clean brightness of a properly developed light roast.

**Roast log indicators:**
- Development time < 60 seconds
- DTR < 0.15
- Drop time while first crack still audible (or seconds after it ended)
- Weight loss % below expected range for target level

**CB-101 specific causes:**
- Dropped too early — mistook the first few isolated pops of pre-crack for the start of first crack, then dropped at "FC end" that was actually still the beginning
- Nervousness around first crack; dropping before the rolling phase fully completes
- Target roast level was light, but pulled before actual light roast window

**Root mechanism:**
The development phase is when the chemical transformations responsible for sweetness, body, and pleasant acidity are completed. Cut it short and those reactions are incomplete. The organic acids that create harsh sourness haven't been transformed. The sugars haven't caramelized fully.

**Key distinction from properly developed light roast:** A properly developed light roast has clean, bright acidity — citric, lively, pleasant. An underdeveloped roast has harsh, rough sourness — aggressive, drying, unpleasant. If you're not sure which you have, cup it next to a known properly-developed light roast.

**Fix:**
Let first crack fully roll and complete before starting your development clock. A full rolling first crack lasts 1–3 minutes. Don't count elapsed time from the first pop — count from when the crack is clearly rolling. Minimum 45 seconds of development after the rolling crack ends.

---

### Scorched / Tipped

**Cup symptoms:** Harsh bitterness distinct from dark-roast bitterness — sharper, more acrid. Rubber, plastic, or phenolic notes. Can coexist with otherwise light-roast character (light color but bad taste). The harshness hits immediately and doesn't fade.

**Roast log indicators:**
- Drying ratio < 0.25 (yellowed very fast)
- Total roast time short
- High initial temperature setting
- Observations noting harsh or acrid smell early

**CB-101 specific causes:**
- Initial temperature set too high
- Charging into a machine that's still hot from a previous roast
- Second batch without the machine cooling to charge temperature

**Root mechanism:**
The outer bean surface is being carbonized before the interior moisture has migrated. The bean tip and surface take on localized char while the interior may still be underdeveloped. The result is simultaneous scorch and underdevelopment — the worst of both worlds.

**Visual check:**
After cooling, look for beans with dark spots or tips while the rest of the bean is lighter. Those are tipped beans. If you see them, expect scorch notes in the cup.

**Fix:**
Lower initial temperature setting. Allow machine to cool fully between batches. A starting temperature of 390–410°F is typically a safe range on the CB-101 for most green beans.

---

### Overdeveloped

**Cup symptoms:** Bitter, ashy, smoky, harsh. Roast character dominates everything — origin character is absent. Heavy body but not pleasant. The bitterness lingers on the back of the throat. Can taste burnt.

**Roast log indicators:**
- Development time > 180 seconds after FC end
- DTR > 0.25
- Weight loss significantly above expected range
- Second crack onset recorded, or observations noting second crack beginning

**CB-101 specific causes:**
- Lost track of time during development phase
- Temperature maintained too high through development
- Distraction at the critical drop moment

**Root mechanism:**
Pyrolysis — the thermal breakdown of the bean's cellular structure. The complex aromatic compounds built during Maillard and development are being destroyed. Carbon compounds from combustion replace them.

**Fix:**
Be at the machine and ready to drop from first crack onset. Have your development target time written down before you start. Set a physical timer at FC end.

---

### Uneven Roast

**Cup symptoms:** Contradictory — simultaneously sour and bitter, both underdeveloped and overdeveloped notes. The cup tastes confused rather than balanced. Complexity that reads as chaos rather than nuance.

**Roast log indicators:**
- Numbers may look normal — this defect hides in the log
- Sensory notes during roast may mention inconsistent color or patchy browning

**CB-101 specific causes:**
- Batch size too large for the drum to tumble freely (> 260g green is risky)
- Mixed bean density in the batch (e.g., blending beans of very different origins or sizes)
- Beans loaded unevenly

**Root mechanism:**
Some beans developed faster than others. When you drop at the right time for the "average" bean, some are overdeveloped and some are underdeveloped. The cup blends both.

**Visual check:**
After roasting, spread beans on a flat surface and look for color variation. Some beans noticeably lighter or darker than the rest is the signature of an uneven roast.

**Fix:**
Reduce batch size. Sort beans for obvious size/density outliers before roasting.

---

### Quakers

**Cup symptoms:** Papery, peanut-like, or bland off-flavor that doesn't fit the roast profile. The cup is mostly good but has a flat, starchy, muted note that interrupts the finish or body.

**Roast log indicators:**
- Not detectable from numbers alone
- Visual inspection at drop: pale/underdeveloped beans visible among the normally-roasted batch

**Root mechanism:**
Quakers are immature green beans that lack the sugars and structure needed for proper Maillard development. They roast lighter than the surrounding beans because there's nothing to develop. They taste like starchy, papery, raw grain because that's essentially what they are.

**Fix:**
Sort green beans before roasting. Look for pale, shrunken, or hollow-looking beans and discard. Ask your supplier about defect counts — high-quality specialty green should have very low defect rates.

---

## Cross-Reference: Cup → Roast Cause

Start with what you're tasting and find the likely roast cause.

| Tasting | Primary suspect | Secondary suspect | Check in log |
|---|---|---|---|
| Sour, harsh, rough | Underdeveloped | Scorched (if also harsh/acrid) | DTR < 0.15, development_time_s < 60 |
| Flat, papery, dull | Baked | — | Total time > 16 min, drying ratio > 0.50 |
| Grassy, vegetal, raw grain | Underdeveloped | Quakers | development_time_s, visual check for quakers |
| Bitter, ashy, smoky | Overdeveloped | — | DTR > 0.25, weight_loss_pct high |
| Harsh, acrid, rubbery | Scorched | Overdeveloped | Drying ratio < 0.25, initial temp high |
| No sweetness, no acidity, no character | Baked | — | Look for Maillard stall in observations |
| Simultaneous sour + bitter | Uneven roast | — | Color variation visible at drop? Batch size? |
| Peanut / papery note in otherwise good cup | Quakers | — | Visual: pale beans in batch? |
| Clean brightness, pleasant citrus/fruit | Properly developed light roast | Not a defect | DTR 0.18–0.22, development 60–90 sec |

---

## Cross-Reference: Roast Decision → Cup Effect

Start with a decision you made and understand what it's likely to produce.

| Decision | Likely cup effect |
|---|---|
| Reduced temp during Maillard phase | Bake risk; flat, papery, dull |
| Dropped at first pop (before rolling crack) | Underdeveloped; harsh sourness |
| Dropped 90 sec after FC end for medium | Sweet, balanced — target result |
| Charged into hot machine (second batch) | Scorch risk in drying; harsh bitterness |
| Extended development to 3+ min | Overdeveloped; bitter, roast-dominant |
| Batch over 260g in CB-101 | Uneven roast; contradictory cup |
| Maintained high temp through development | Risk of pushing past target level quickly |
| Reduced temp at FC onset (intentional) | Slower development; more control over stop point |

---

## Comparing a Roast to Its Previous Version

When you're trying to understand whether an adjustment worked, compare these fields across the two roast logs:

1. **DTR** — did it change in the direction you intended?
2. **Weight loss %** — proxy for roast level; should be consistent if targeting the same level
3. **Total roast time** — major change here means something else changed too (ambient temp, batch size, machine state)
4. **Yellowing time** — if this shifted significantly, the drying phase changed, which affects everything downstream
5. **Temperature adjustments** — were the two roasts controlled the same way, or did you intervene differently?
6. **Cupping ratings** — acidity, sweetness, body across both roasts; did the change move them in the predicted direction?

If the numbers look the same but the cup tastes different, look at:
- Ambient temperature (cold days produce different roast behavior)
- Green bean age (freshness matters; older green is flatter)
- Machine state (was it fully pre-heated both times?)
- Rest time before cupping (same rest hours for both?)
