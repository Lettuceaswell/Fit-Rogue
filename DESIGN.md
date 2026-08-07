# Fit-Rogue — Design Doc (v0.2.1)

A single-file, mobile-first, roguelike life-simulator. You play one life from age 19 to 67 in 2-year rounds, allocating a shared energy budget across work, training, and diet while managing an ever-rising stress floor. The game ends in one of two states: Crushed by stress, or retired well.

Implementation: `index.html`, plain HTML/CSS/JS, no build step, no dependencies. Prior iterations kept in `versions/` (v0.1.1, v0.1.2, v0.2).

## Premise / Fantasy

Not a fitness app — a **resource-management roguelike** wearing a "fitness" skin. There are two currencies, Energy which you wish you could spend more of, and Cash which you wish you could save. Stress is the run-ending resource: it creeps up naturally each round, applying pressure and spikes with a "boss" every 3rd round, and you must reduce it to zero or beyond with your 'score' each round.

## Core Loop

One **round = 2 years**, and a run is `CONFIG.ROUNDS` (24) rounds long, aging the player from 19 to 67. Each round has three sequential stages:

1. **Status Update** — Fill energy in accordance with your currently held diet. Players always start with "Convenience food" diet card. The Stress Counter is set 
2. **Allocate** — set work dial (overtime vs. time off) and training intensity within the energy budget; see a live projection of the round's outcome before committing.
3. **Upgrade** — spend saved dollars on permanent round-over-round bonuses (job promotion, spouse, gym) before advancing.

Committing a round (`commitRound`) resolves all deltas at once — dollars, bodyfat, muscle, stress — and appends a one-line summary to a running life log. The player then either continues to Upgrade, or the run ends immediately if stress has consumed the usable energy bar entirely.

## The Central Mechanic: The Energy Bar

Everything routes through one number: **usable energy** = `barMax() - stress`.

- `barMax` = `BASE_BAR (100) + muscle` — building muscle literally grows your capacity to take on life.
- `stress` eats the bar from the left, permanently, until relieved.
- Every activity (job, overtime, training) *spends* energy from what's left.
- Diet supplies *intake* energy; the balance between intake and spend determines whether you gain or lose bodyfat.

This produces a three-segment bar visualization in the UI: **stress** (fixed, dark) → **spend** (this round's plan, mid-grey) → **slack** (unspent, light — flows into bodyfat if positive, or represents shortfall if the plan can't be paid for).

Stress has a **base floor** that rises every decade (`STRESS_BASE_START` 6, `+2` per 2-round "decade"), so the bar's usable width shrinks over the whole run regardless of player choices — this is the game's clock. Training and spouse choices are the primary levers to push back against it. If usable energy ever drops to or below the base job's energy cost, the run ends in the `stress` (loss) outcome.

## Resources

| Resource | Role |
|---|---|
| **Energy** (bar) | The per-round budget; shrinks as stress rises, grows as muscle rises. Spent by job, overtime, training; supplied by diet. |
| **Stress** | Never decreases passively. Rises from overtime, high-intensity training, bodyfat outside the healthy band, and the decade floor. Reduced only by light/moderate training and certain spouses. This is the loss condition. |
| **Dollars** | Earned from job pay + overtime − time-off penalty; spent on diet cost and one-time upgrades (job tier, spouse, gym). Not itself a loss condition, just a gate on upgrades. |
| **Bodyfat %** | Moves with the energy balance (surplus → up, deficit → down) each round. Has a healthy band (`BF_LOW` 10 – `BF_HIGH` 20); outside it adds a flat stress penalty every round. |
| **Muscle** | Widens the energy bar (the core positive-feedback loop). Requires *both* a protein-positive diet and a non-negative energy balance to grow on a training round; always decays by `-3` if training is skipped entirely (`None` tier). |

## Choices Per Round

**Diet** (pick one, costs money, sets energy intake):
- Fast Food — cheap, huge energy, no protein (can't build muscle).
- Healthy — expensive, moderate energy, has protein, no prep cost.
- Cook — cheap, moderate energy, has protein, but costs upfront energy to prepare.

**Work dial** (single slider, `-TIMEOFF_MAX..+OVERTIME_MAX`):
- Overtime: costs energy and adds stress, pays extra.
- Time off: frees energy, but costs money (lost pay) and adds stress (implying idle time is *not* restful in this model — a deliberate tension, see Open Questions).
- Overtime is capped dynamically by remaining energy room (`clampWork`), not just a fixed max.

**Training** (pick one tier, gated by remaining energy via `maxTrainTier`):
- None — free, but muscle decays (engine atrophies from disuse).
- Light — cheap, relieves stress, holds muscle.
- Moderate — relieves stress, builds muscle (needs protein + surplus).
- High — costs more, *adds* stress, builds muscle fastest.

The Allocate screen shows a live projection panel (`.proj`) computing energy balance, resulting bodyfat, muscle, net dollars, and post-round stress *before* the player commits — this is the "read the board before you play the card" moment central to the roguelike feel.

**Upgrades** (Upgrade stage, one-time purchases, dollars-gated):
- Job promotion (3 tiers: Warehouse → Supervisor → Manager) — more pay, but also higher fixed energy cost and stress per round.
- Spouse (pick one of three, permanent): Supportive (flat stress relief), Driven (extra income, some stress), Athletic (cheaper training).
- Gym (2 tiers, stack via upgrade path): Home setup (cheaper training), Full gym (bonus muscle on build rounds, further training discount).

These are irreversible per-run choices that shape which strategy (income-heavy, stress-mitigation, or muscle-engine) the rest of the run leans into — the closest thing this design has to a "build" in the deckbuilder sense.

## End States

Computed in `nextRound`/`commitRound`:
- **Stress (loss)** — usable energy can no longer cover even the base job cost. Ends immediately, any round.
- **Retired Well (win)** — reaches round 10 (age 70) with bodyfat inside the healthy band *and* muscle ≥ `HEALTHY_MUSCLE_MIN` (16).
- **Retired Unwell (partial loss)** — reaches age 70 but outside the healthy band or under the muscle threshold.

## Visual / UX Design

- Monochrome, monospace, "quantified self" aesthetic (`ui-monospace`, greys, single accent of near-black). No color signaling beyond a single red (`.bad`/`.stat .v.bad`) for out-of-band stats.
- Single-column, `max-width:520px`, built mobile-first (sticky header bar, large 48px tap targets, no hover-dependent affordances).
- The stress/spend/slack bar is the persistent header on every screen — the player's "board state" is always visible while making decisions lower on the page.
- Card-based selection (diet, training, upgrades) rather than dropdowns/forms, consistent with a roguelike-choice visual language.
- A running "Life so far" log (last 6 rounds) gives retrospective narrative without needing a separate history screen.
- No animation/juice beyond CSS transitions on the bar segments; deliberately flat and numeric — the game trusts its numbers to carry tension rather than presentation.

## Tuning Surface

All balance numbers live in one `CONFIG` object at the top of the script, with an explicit comment convention: *"change ONE per playtest session."* This is the project's primary iteration mechanism — no separate data files, no build step, just direct constant edits and re-testing in browser. Versions are snapshotted into `versions/vX.Y.html` as balance/structure passes are completed, which is also the changelog.

## Known Open Questions / Rough Edges (as of v0.2)

- Time off increasing stress is a strong, slightly counterintuitive design choice (rest costs you) — reinforces the "everything costs something" theme but may need a clearer in-UI justification (e.g. flavor text) so it doesn't read as a bug.
- No difficulty/seed variance yet — every run faces the same stress-floor schedule and upgrade costs, so replay variety currently comes only from player strategy, not randomness. Worth deciding if randomized events belong in a future iteration or if the design intentionally stays deterministic ("roguelike" here means run-based/permadeath structure, not randomized content).
- No persistence between runs (no meta-progression) — each run starts from `newRun()` with identical starting stats.
