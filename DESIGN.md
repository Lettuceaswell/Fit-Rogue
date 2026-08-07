# Fit-Rogue — Design Doc (v0.2.1)

A single-file, mobile-first, roguelike life-simulator. You play one life from age 19 to 67 in 2-year rounds, allocating a shared energy budget across work, training, and diet while managing an ever-rising stress floor. The game ends in one of two states: Crushed by stress, or retired well.

Implementation: `index.html`, plain HTML/CSS/JS, no build step, no dependencies. Prior iterations kept in `versions/` (v0.1.1, v0.1.2, v0.2).

## Premise / Fantasy

Not a fitness app — a **resource-management roguelike** wearing a "fitness" skin. There are two currencies, Energy which you wish you could spend more of, and Cash which you wish you could save. Stress is the run-ending resource: it creeps up naturally each round, applying pressure and spikes with a "boss" every 3rd round, and you must reduce it to zero or beyond with your 'score' each round.

## Core Loop

One **round = 2 years**, and a run is `CONFIG.ROUNDS` (24) rounds long, aging the player from 19 to 67. Each round has three sequential stages:

1. **Status Update** — Fill energy in accordance with your currently held diet. Players always start with "Convenience food" diet card. The Stress Counter is set according to round escalation and possible modifiers from previous round.
2. **Cards** Play any held cards to modify your build. Also offer Overtime slider which will yeild more cash per hour, but raise stress beyond expected next round. Also Offer PTO button then confirmation button - PTO will skip any money earned, but will put a dent in this round's stress.
3. **Train** You press Train to confirm your attempt, and stats and modifiers are calculated to determine wether the score is enough to beat the stress level.


Stress has a **base floor** that rises every decade so the bar's usable width shrinks over the whole run regardless of player choices — this is the game's clock. If Training doesn't reduce stress to zero successfully, that's the death of the run.

## Resources

| Resource | Role |
|---|---|
| **Energy** (bar) | Must-spend resource; remaining energy will determing body fat % at start of next round. less than or equal to 16% energy will result in 5% body fat. from 17-33% energy will result in 10% Body fat. from 34-50% energy will result in 20% body fat. from 51-66% energy will result in 30% bf. from 67-83% energy will result in 40% bf. from 84-100% energy will result in 50% bf |
| **Stress** | Must be reduced to zero every round. Rises naturally by round, and from other sources like overtime, bodyfat outside the healthy band, and other. This is the loss condition. |
| **Dollars** | Earned from job pay + overtime; spent on all card upgrades (job tier, spouse, gym). Not itself a loss condition, but debt is allowed, and it will raise stress level. |
| **Body fat %** | 6 stat levels: 5%, 10%, 20%, 30%, 40%, 50%. Moves with the energy balance of the previous round as previously indicated. Has a healthy band (10% – 20%); outside it adds a flat stress penalty every round. |
| **Muscle** | 6 stat levels: 1, 2, 3, 4, 5, 6. A higher muscle stat Helps you to burn excess energy more aggressively. Unless you have a card allowing it, You cannot gain a muscle level without gaining 1 or more Body fat Levels at the same time. If one or more fat levels are gained, the muscle level will be boosted by one stat level.

## Choices Per Round

**Work**
- Overtime: spends energy and adds stress, pays extra.
- Time off: doesn't use energy, but has immediate stress reduction effect. The resultant Stress Delta is shown before you confirm.
- Overtime is capped dynamically by remaining energy room, not just a fixed max.

**Training**
- Dynamically grab the energy slider and slide it to desired training output.
The Allocate screen shows live for resultant 'next round bodyfat', net dollars, and post-round stress *before* the player commits — this is the "read the board before you play the card" moment central to the roguelike feel.

**Upgrades**
- Job promotion 
- Spouse
- Gym 

These are always replaceable when a valid card is played to replace them.
