# Axie Classic -- Morale Stat Revamp Proposals

## Current State of Morale

Morale is one of four Axie stats (HP, Speed, Skill, Morale) and currently governs:

1. **Critical hit chance** -- Higher morale = higher crit probability (~1% per morale point, exact formula unclear to players)
2. **Critical hit damage** -- Morale influences crit multiplier, capped at 200%
3. **Last Stand entry** -- Higher morale = more likely to enter Last Stand instead of dying outright (threshold based on overkill vs remaining HP * morale factor)
4. **Last Stand ticks** -- Higher morale = more Last Stand turns before death

| Stat | Base Range | Beast Base | Pure Beast (6 parts) | Plant Base | Pure Plant (6 parts) |
|------|-----------|------------|----------------------|------------|----------------------|
| Morale | 27-43 | 43 | 61 | 31 | 31 |

### Why Morale Underperforms

- **Crit chance is too low** to be a reliable strategy. Even at 61 morale, crits feel random rather than consistent.
- **200% crit cap** limits the payoff when crits do land.
- **Last Stand provides 1-3 ticks** across the entire morale range. The difference between 31 and 61 morale feels marginal (~1-2 extra ticks), not transformative.
- **Morale has no effect on gameplay until the Axie is about to die** (Last Stand) or gets lucky (crit). It's the least "felt" stat in moment-to-moment play.
- **Speed, HP, and Skill all have constant, visible impact.** Morale is probabilistic and backloaded.

---

## Proposal 1: Exponential Last Stand Scaling

### Core Idea
Make Morale scale non-linearly for Last Stand ticks so high-morale Axies have dramatically more Last Stand bars, creating a true "second phase" of combat.

### Current vs Proposed Last Stand Ticks

| Morale | Current Ticks | Proposed Ticks |
|--------|--------------|----------------|
| 27 | 1 | 1 |
| 31 | 1-2 | 1 |
| 35 | 1-2 | 2 |
| 40 | 2 | 2 |
| 43 | 2 | 3 |
| 45 | 2 | 3 |
| 50 | 2-3 | 4 |
| 55 | 2-3 | 5 (+1 bonus) |
| 61 | 2-3 | 7 (+2 bonus) |

### Formula
```
base_ticks = floor((morale - 25) / 5)
bonus_ticks = floor(max(0, morale - 52) / 3)
total_ticks = base_ticks + bonus_ticks
```

### Design Intent
A pure Beast (61 Morale) getting 7 Last Stand ticks vs a Plant's 1 makes Morale feel like it matters. The Beast doesn't just die slower -- it has an entire second phase of combat where it can draw, combo, and crit. This directly solves the "I died before I drew my combo" problem.

### Last Stand Entry Threshold
Increase the HP threshold for entering Last Stand based on morale:
- Current: Enter Last Stand if overkill damage < `remaining_hp * (morale / some_divisor)`
- Proposed: Enter Last Stand if overkill damage < `remaining_hp * (morale / 80)`. At 61 morale, you enter Last Stand if overkill is < 76% of remaining HP. At 31 morale, threshold is < 39%.

### Interaction with Existing Mechanics
- **Chill** remains the primary counter (prevents Last Stand entry entirely)
- **Lethal** remains effective (kills through Last Stand)
- **High burst** that exceeds the entry threshold still bypasses Last Stand
- Last Stand HP per tick stays the same (small), so high-tick Last Stand is about *actions*, not *tankiness*

### Risks & Mitigations
| Risk | Severity | Mitigation |
|------|----------|------------|
| Beast/Bug Last Stand stalling becomes oppressive | High | Chill and Lethal must remain accessible across all classes |
| Games drag on with long Last Stand sequences | Medium | Last Stand tick HP remains low; each tick is ~1 action, not a full survival phase |
| Pure morale builds become mandatory for Beast | Low | This is acceptable -- morale IS Beast's class identity |
| Low-morale Axies feel like they lost a feature | Medium | 1 tick is still standard; this doesn't remove Last Stand from low-morale Axies |

---

## Proposal 2: Morale as a Resource (Rage System)

### Core Idea
Convert Morale from a passive stat into an active resource called "Rage" that high-morale Axies generate and spend on powerful combat effects. This makes Morale a constant decision point, not a background probability.

### Rage Generation
```
rage_per_round = floor(morale / 10)
max_rage_pool  = morale
rage_decay     = 50% of unspent rage at end of round (floor)
```

| Morale | Rage/Round | Max Pool | Rounds to Fill |
|--------|-----------|----------|----------------|
| 27 | 2 | 27 | Never (decay) |
| 31 | 3 | 31 | Never (decay) |
| 35 | 3 | 35 | Never (decay) |
| 43 | 4 | 43 | ~5 rounds |
| 50 | 5 | 50 | ~5 rounds |
| 61 | 6 | 61 | ~5 rounds |

### Rage Abilities (Spend Options)

| Ability | Rage Cost | Effect |
|---------|-----------|--------|
| Savage Strike | 8 | Next card this Axie plays is a guaranteed critical hit |
| Ferocity | 12 | Next card this Axie plays deals +50% damage |
| Defiant Stand | 15 | This Axie enters Last Stand when it would die this round (guaranteed, ignores overkill) |
| Undying Fury | 20 | Gain +1 Last Stand tick |
| Blood Surge | 25 | All Beast cards this Axie plays this round have +30% damage and guaranteed crit |

### Usage Rules
- Rage is spent *before* playing cards in the action phase (during the planning phase)
- Multiple Rage abilities can be activated per round if the Axie has enough Rage
- Rage is visible to the opponent (informed counterplay)
- Rage does not carry across Last Stand (resets to 0 on Last Stand entry)

### Design Intent
Low-morale Axies (Plant at 31) generate 3 Rage/round but lose 50% to decay, so they can occasionally afford Savage Strike (8 cost) every ~3 rounds. High-morale Axies (Beast at 61) generate 6/round and can realistically build toward Blood Surge (25 cost) over 4-5 rounds. The stat creates meaningful decisions every single round.

### Risks & Mitigations
| Risk | Severity | Mitigation |
|------|----------|------------|
| Complexity increase for new players | High | Rage abilities are optional; ignoring them is suboptimal but not crippling |
| UI/UX burden (new resource bar, spend interface) | High | Requires client work in unity-axie-classic; significant engineering effort |
| Balance across all classes | High | Decay mechanic ensures low-morale Axies can't stockpile; costs tuned so only 45+ morale enables consistent use |
| Interaction with energy system | Medium | Rage is separate from energy; no conversion between them |

---

## Proposal 3: Morale-Driven Crit Overhaul

### Core Idea
Keep Morale's current roles but dramatically overhaul the crit system so high-morale Axies crit reliably and those crits have meaningful, distinct impact. Make crits a core gameplay mechanic, not an occasional bonus.

### New Crit Chance Formula
```
crit_chance = (morale - 27) * 2.5%
```

| Morale | Current Crit % (approx) | New Crit % |
|--------|------------------------|------------|
| 27 | ~5% | 0% |
| 31 | ~8% | 10% |
| 35 | ~10% | 20% |
| 43 | ~15% | 40% |
| 50 | ~18% | 57.5% |
| 55 | ~20% | 70% |
| 61 | ~23% | 85% |

### New Crit Damage Formula
Remove the 200% cap. Replace with morale-scaled crit damage:
```
crit_damage_multiplier = 1.0 + (morale / 200)
```

| Morale | Current Crit Damage | New Crit Damage |
|--------|--------------------|-----------------| 
| 27 | Up to 200% | 113.5% (×1.135) |
| 31 | Up to 200% | 115.5% (×1.155) |
| 43 | Up to 200% | 121.5% (×1.215) |
| 61 | Up to 200% | 130.5% (×1.305) |

**Note:** The crit damage multiplier is lower per-hit than the current 200% cap, but it procs far more often. Expected damage over multiple attacks is higher for high-morale Axies and more consistent.

### Expected Damage Comparison (per 100 ATK card)

| Morale | Current Expected | New Expected | Change |
|--------|-----------------|--------------|--------|
| 27 | ~105 | 100.0 | -5% |
| 31 | ~108 | 101.6 | -6% |
| 43 | ~115 | 108.6 | -6% |
| 61 | ~123 | 125.9 | +2% |

The expected damage is similar, but the *variance* is dramatically reduced for high-morale Axies. A 61-morale Beast crits 85% of the time -- it's nearly guaranteed, not a dice roll.

### Crit Side Effect: "Shaken" Debuff
When a critical hit lands, apply **Shaken** to the target for 1 turn:
- Target's shield effectiveness is reduced by 25%
- Does not stack (refreshes duration)

This gives crits utility beyond damage. A Beast critting consistently applies permanent Shaken to its target, weakening shield-heavy tanks.

### Risks & Mitigations
| Risk | Severity | Mitigation |
|------|----------|------------|
| 85% crit chance feels deterministic, removes excitement | Medium | 15% non-crit keeps some variance; the consistency IS the point |
| Lower crit multiplier may disappoint players used to big crit numbers | Medium | Communicate as "reliable crits" vs "lottery crits"; total damage output is similar or higher |
| Shaken debuff may be too strong against Plant/Reptile shield walls | Medium | 25% reduction is moderate; test at 20% if needed |
| Multi-hit cards with high crit chance could be broken | High | Multi-hit cards should roll crit per-hit; individual hits are weaker, so per-hit crit is fine |
| Bug/Mech classes also have high morale | Low | Bug 41 = 35% crit, Mech derives from parents. Both are fair users of the system |

---

## Proposal 4: Morale as Tenacity (Scaling Damage Reduction)

### Core Idea
Redefine Morale as "fighting spirit" -- a stat that makes high-morale Axies progressively harder to kill as they take damage. Instead of being a crit/Last Stand stat, Morale provides scaling damage reduction based on missing HP, creating a "cornered animal" fantasy.

### Damage Reduction Formula
```
hp_percent = current_hp / max_hp
dr_scaling = (1.0 - hp_percent) * (morale / 100)
damage_reduction = min(dr_scaling, 0.65)  // hard cap at 65%
```

### Damage Reduction by HP Threshold

| HP % | 27 Morale (Plant) | 35 Morale (Aqua) | 43 Morale (Beast base) | 61 Morale (Pure Beast) |
|------|-------------------|-------------------|------------------------|------------------------|
| 100% | 0% | 0% | 0% | 0% |
| 75% | 6.8% | 8.8% | 10.8% | 15.3% |
| 50% | 13.5% | 17.5% | 21.5% | 30.5% |
| 25% | 20.3% | 26.3% | 32.3% | 45.8% |
| 10% | 24.3% | 31.5% | 38.7% | 54.9% |
| Last Stand | 16.2% | 21.0% | 25.8% | 36.6% |

**Last Stand DR** is reduced to 2/3 of the 10% HP value to prevent infinite stalling.

### Interaction with Existing Stats
- **HP stat** still determines raw health pool. Morale doesn't increase HP, it makes each HP point worth more at low health.
- **Shield** applies before DR. Damage hits shield first, then remaining damage is reduced by DR, then applied to HP.
- **True damage** (e.g., Poison ticks) ignores DR.
- **Percentage-based damage** ignores DR (if any exists).

### Combat Feel
A pure Beast at 25% HP takes 45.8% less damage from attacks. A 100-damage card only deals 54 damage. This gives the Beast time to draw its combo, find its burst, and actually execute its gameplan. The lower the Beast's HP, the harder each point of HP is to remove -- until Last Stand, where DR drops to prevent stalling.

### Comparison to Current Last Stand Approach
| Aspect | Current (Last Stand ticks) | Proposed (Tenacity DR) |
|--------|---------------------------|------------------------|
| When it matters | Only at 0 HP | Entire low-HP range (50% and below) |
| Counterplay | Chill/Lethal bypass | High sustained damage, true damage, or burst past DR |
| Skill expression | Low (passive) | Medium (HP management, deciding when to shield vs take hits) |
| Visual clarity | Clear (Last Stand bars) | Needs DR indicator on Axie |

### Risks & Mitigations
| Risk | Severity | Mitigation |
|------|----------|------------|
| Stacking with shields makes Beast unkillable at low HP | High | Shield applies before DR; also cap DR at 65% |
| True damage/Poison becomes the only way to kill Beast | Medium | DR only applies to card damage; Poison ignores it. This is intended counterplay. |
| Low-morale Axies (Plant) barely notice the system | Low | Intentional. Plant has HP; Beast has tenacity. Different survival tools. |
| Hard to communicate DR to players | Medium | Show a "Tenacity" shield icon with percentage when DR > 10% |
| Healing + DR could be abusive | Medium | Beast has very limited healing access; monitor if new healing cards change this |

---

## Proposal 5: Morale Aura (Team-Wide Morale Influence)

### Core Idea
Make Morale a partially team-shared stat. High-morale Axies project a "Morale Aura" that increases their allies' effective Morale for crit chance and Last Stand calculations. When a high-morale Axie dies, the team suffers a "Morale Break" debuff. This turns Morale into a strategic team-building and positioning stat.

### Aura Mechanics
```
aura_value = floor(morale / 5)
effective_morale = base_morale + sum(ally_auras)
// Aura does NOT count toward the projector's own morale
// Aura does NOT contribute to further aura calculations (no recursion)
aura_cap = 15  // max morale received from allies
```

### Aura Values by Class

| Class | Base Morale | Aura Projected | Example: 2 Allies Receive |
|-------|------------|----------------|---------------------------|
| Plant | 31 | 6 | +6 each |
| Aquatic | 35 | 7 | +7 each |
| Bird | 35 | 7 | +7 each |
| Beast | 43 | 8 | +8 each |
| Bug | 41 | 8 | +8 each |
| Pure Beast (6 parts) | 61 | 12 | +12 each |
| Pure Bug (6 parts) | 53 | 10 | +10 each |

### Team Composition Example: Beast / Plant / Aquatic

| Axie | Base Morale | Aura Received | Effective Morale |
|------|------------|---------------|------------------|
| Pure Beast (61) | 61 | Plant 6 + Aqua 7 = 13 | 74 |
| Pure Plant (31) | 31 | Beast 12 + Aqua 7 = 15 (capped) | 46 |
| Pure Aquatic (35) | 35 | Beast 12 + Plant 6 = 15 (capped) | 50 |

The Beast buffs its entire team's crit chance and Last Stand. The Plant goes from functionally no-crit (31 morale) to moderate crit potential (46 effective morale).

### Morale Break (Death Penalty)
When an Axie with 45+ base Morale dies:
- All surviving allies receive **Demoralize** for 1 round
- Demoralize: -10 effective Morale (reduces crit chance and Last Stand threshold)
- Demoralize cannot stack (if multiple high-morale allies die, it refreshes duration)
- Demoralize does NOT apply if the dying Axie enters Last Stand (Break only triggers on true death)

### Strategic Implications
1. **Protecting high-morale Axies becomes important.** The Beast in the back is an aura battery -- losing it early cripples the team's crit potential.
2. **Killing the Beast first is a valid counter-strategy.** Remove the aura AND apply Morale Break.
3. **Triple-Beast comps** get massive aura stacking: each Beast at 61 morale projects 12, receiving 24 from allies (capped at 15), putting effective morale at 76. This needs monitoring.
4. **Mixed teams** still benefit meaningfully. A single Beast giving +12 to two allies is impactful without being oppressive.

### Risks & Mitigations
| Risk | Severity | Mitigation |
|------|----------|------------|
| Triple-Beast/Bug comps become oppressive via aura stacking | High | Aura cap of 15 prevents runaway. Monitor and adjust cap if needed. |
| Hard to communicate effective morale to players | Medium | Show aura icon with +value next to morale stat. Highlight effective morale in UI. |
| Morale Break feels-bad when losing is already bad | Medium | 1-round duration limits impact. Only triggers on true death, not Last Stand. |
| Aura makes team-building too "solved" (always include a high-morale Axie) | Medium | Aura competes with other team-building considerations (class coverage, card synergy). 15 cap limits extreme builds. |
| Recursive aura edge cases | Low | Explicitly non-recursive. Aura is calculated from base morale only. |

---

## Comparison Matrix

| Criteria | 1. Exponential LS | 2. Rage Resource | 3. Crit Overhaul | 4. Tenacity DR | 5. Morale Aura |
|----------|-------------------|------------------|-------------------|----------------|----------------|
| **Solves "morale doesn't feel impactful"** | Yes (Last Stand) | Yes (constant) | Yes (crits) | Yes (survival) | Yes (team-wide) |
| **When morale matters** | At death | Every round | Every attack | Below 50% HP | While alive |
| **Implementation complexity** | Low | High | Medium | Medium | Medium-High |
| **Client (Unity) changes needed** | Minor (LS bar count) | Major (new resource UI) | Minor (crit VFX, Shaken icon) | Medium (DR indicator) | Medium (aura display) |
| **Server (Rust) changes needed** | Low (formula tweak) | High (new system) | Medium (formulas + Shaken) | Medium (DR calculations) | Medium (aura calculations) |
| **Risk of breaking game balance** | Medium | High | Medium | Medium-High | Medium |
| **Preserves Beast class identity** | Yes | Yes | Yes | Partially (shifts to tank) | Yes (enhances team role) |
| **Skill expression added** | Low | High | Low-Medium | Medium | Medium |
| **Counterplay clarity** | High (Chill/Lethal) | Medium (new system) | Medium (anti-crit) | Medium (true damage/burst) | High (kill the aura source) |
| **Combinable with other proposals** | Yes (with 3, 4, or 5) | Standalone | Yes (with 1 or 5) | Yes (with 1 or 3) | Yes (with 1 or 3) |

---

## Recommended Combinations

If multiple proposals are adopted, these pairings work well together:

1. **Exponential LS (1) + Crit Overhaul (3):** Beast crits reliably AND has a real Last Stand phase. Two different payoffs for high morale at two different points in the game (during combat and at death).

2. **Tenacity DR (4) + Morale Aura (5):** Beast becomes a team anchor -- harder to kill at low HP and buffing allies while alive. Losing the Beast is devastating (lost aura + Morale Break), creating strategic tension.

3. **Exponential LS (1) + Morale Aura (5):** Beast has massive Last Stand AND empowers the team. The aura means even if the Beast enters Last Stand, the team already benefited from its presence.

**Not recommended together:** Rage (2) with anything else -- it's a standalone system redesign that conflicts with layering other morale changes on top.
