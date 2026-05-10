# Staking Economics Simulator — UI Spec (v2)

_Single-page HTML, vanilla JS + Chart.js. Audience = retreat attendees, not researchers. Goal: see what's changing in staking and why. Keep inputs to **the few knobs governance can actually turn**, plus a year selector. Everything else is output._

> Naming: every governance-tunable input is labelled with its **exact pallet identifier** (`BudgetAllocation`, `OptimumSelfStake`, etc.). Tooltip shows the on-chain default and which extrinsic sets it.

---

## Layout

```
┌────────────────────────────────────────────────────────────────────┐
│ HEADER: title + presets (Today / 2.3.0 / mid-2026 / Q4 with pUSD) │
├──────────────────┬─────────────────────────────────────────────────┤
│  INPUTS (left)   │  OUTPUTS (right)                                │
│  ~340px          │                                                 │
│                  │  [DAP flow diagram — drip → pots]               │
│  [Year]          │                                                 │
│  [DAP split]     │  [KPI strip: cost to protocol + 3 numbers]      │
│  [Validator]     │                                                 │
│  [Modes]         │  [Chart 1: Total supply — legacy vs Ref 1710]   │
│                  │  [Chart 2: Validator + nominator income / yr]   │
└──────────────────┴─────────────────────────────────────────────────┘
```

---

## INPUTS

### Group 1 — Year selector

Single slider. Drives the issuance curve, total issuance at that point in time, and projected income.

| Input | Default | Range |
|---|---|---|
| Year | 2026 | 2026 – 2036 |

Issuance constants (`HARD_CAP_TARGET`, `BI_ANNUAL_RATE`, `MARCH_2026_TI`, etc.) are **fixed**, not exposed. Total issuance for the selected year is computed and displayed as a read-only field below the slider:

> **Total issuance in 2028:** 1.78B DOT _(current: 1.683B)_

### Group 2 — DAP budget split (`BudgetAllocation`)

Three sliders, must sum to 100%. Stacked-bar preview at top of group. Drag one → others auto-rebalance.

| Slider | `BudgetKey` | Default (PAH 2.3.0) | Set by |
|---|---|---|---|
| Buffer / treasury | `b"buffer"` | 15% | `dap.set_budget_allocation` |
| Staker rewards | `b"staker_rewards"` | 85% | same |
| Validator incentive | `b"validator_incentive"` | 0% | same |

When **pUSD mode** (Group 4) is on, a 4th slider for operational costs appears and the others scale down proportionally.

### Group 3 — Validator earnings probe

Lets the user pick one hypothetical validator and see what they make. **No curve sliders** — `OptimumSelfStake` / `HardCapSelfStake` / `SelfStakeSlopeFactor` stay at their PAH-shipped defaults (T=30k, C=100k, k=50% — applied internally).

| Input | Default | Range | Notes |
|---|---|---|---|
| Validator self-stake | 30,000 DOT | 0 – 1,000,000 | Used for both nominator-style reward and self-stake incentive payout. |
| Validator commission | 5% | 0 – 100% | Paid on top of own-exposure share. |
| Backed-by stake (nominators) | 2,000,000 DOT | 0 – 10,000,000 | Total stake nominators point at this validator. |

Output (rendered next to inputs, always visible):

> **This validator earns / year:** 4,210 DOT + $24,000 pUSD
> &nbsp;&nbsp;• as nominator-style (own + commission): 3,800 DOT
> &nbsp;&nbsp;• as self-stake incentive: 410 DOT
> &nbsp;&nbsp;• as operational pUSD: $24,000 (when enabled)

### Group 4 — Mode toggles

- ☑ **DAP enabled** (off = legacy mint-on-payout, slashes destroyed, single 85/15 split)
- ☐ **Self-stake incentive active** (off ⇒ `validator_incentive` allocation forced to 0)
- ☐ **pUSD operational stream (Q4)** — adds 4th DAP destination, validators get `$2,000/month` in pUSD

Presets in header drive these toggles + Group 2 sliders together (see "Presets" below).

---

## OUTPUTS

### KPI strip (4 numbers across the top of outputs panel)

1. **Cost to protocol / year** — total DOT minted (= sum of all DAP outflows × price)
   _DOT amount + USD-equivalent at $4 assumed price_
2. **Effective annual inflation** — `yearly_emission / total_issuance` as %
3. **Leaked from DOT economy** — `cost_to_protocol × leak%` in $M/yr (leak = 50% assumed, footnote)
4. **Validators in active set** — fixed at 600 today → 300 future (text only, with date)

### Output 1 — DAP flow diagram (pride of place, top of right column)

SVG. Drip enters from left, splits into recipient pots, with arrow widths scaled live to `BudgetAllocation`. Inflows (slashes / burns / fees) join the buffer from below.

```
                                           ┌──► Buffer (15%)
                                           │     ▲ slashes
                                           │     ▲ burns
   Issuance curve ──[per-block drip]──► DAP├──► Staker rewards (85%)
                                           │
                                           ├──► Validator incentive (0%)
                                           │
                                           └──► Operational pUSD (off)
```

When DAP is **off**, diagram redraws as the legacy flow: era boundary mints to stakers + treasury, slashes go to `/dev/null`. Side-by-side toggle would be nice — start with single live diagram.

### Chart 1 — Total supply, legacy vs current

- X axis: years 2026–2036
- Y axis: total issuance in billions of DOT
- **Two lines, both visible:** legacy curve (Ref 1139 — flat 8% off fixed baseline) and Ref 1710 stepped decay toward 2.1B
- Horizontal dashed line at `HARD_CAP_TARGET = 2.1B`
- Vertical guide at the year selected in Group 1
- Caption shows the gap: e.g. "By 2031 the new curve has minted 480M DOT less"

### Chart 2 — Annual income at the selected year

Bar chart. Two stacked bars side-by-side:

- **A typical nominator** (per 1,000 DOT bonded) — current vs new
- **The probe validator** (Group 3 inputs) — current vs new

Each bar stacked: DOT (commission + own share) | DOT (self-stake incentive) | pUSD-equiv (operational)

This is the slide that sells the talk: "For the same security spend, here's what changes for stakers."

---

## Scenario presets (header buttons)

Each loads a coherent set of mode toggles + DAP split. The year slider stays where the user left it.

| Preset | Sets |
|---|---|
| **Today** | DAP off, legacy curve, no self-stake, no pUSD |
| **PAH 2.3.0 (today's launch)** | DAP on, 15 / 85 / 0, self-stake off, no pUSD |
| **Self-stake enabled** | DAP on, 14 / 80 / 6, self-stake on, no pUSD |
| **Q4 — pUSD live** | DAP on, 12 / 70 / 6 / 12 (buf/stk/inc/pusd), all toggles on |

(Split %s for the latter two are illustrative — real values still TBD by governance. Footnote in UI.)

---

## Formulas (JS)

```js
// 1. Total issuance at year Y (Ref 1710 stepped curve)
function totalIssuance(year, initial=16.74e9, target=2.1e9, rate=0.2628, stepY=2) {
  // initial is in DOT (March 2026 baseline), target = HARD_CAP_TARGET
  const steps = Math.floor((year - 2026) / stepY);
  return target - (target - initial) * Math.pow(1 - rate, steps);
}

// 2. Yearly emission (gap between this year and last)
function yearlyEmission(year) {
  return totalIssuance(year+1) - totalIssuance(year);
}

// 3. DAP outflow per pot
function dapOutflow(year, allocationPct) {
  return yearlyEmission(year) * allocationPct;
}

// 4. Self-stake weight (frozen defaults: T=30k, C=100k, k=0.5)
const T = 30_000, C = 100_000, K = 0.5;
function incentiveWeight(s) {
  if (s === 0) return 0;
  if (s <= T) return Math.sqrt(s);
  if (s <= C) return Math.sqrt(T + K*K*(s-T));
  return Math.sqrt(T + K*K*(C-T));
}

// 5. Validator yearly income
function validatorIncome(year, ownStake, commission, backedBy, totalStake, totalValidators) {
  const stakerPot = dapOutflow(year, allocations.staker_rewards);
  const incentivePot = dapOutflow(year, allocations.validator_incentive);

  // share of staker pot via own + commission on backedBy
  const ownShare = stakerPot * (ownStake / totalStake);
  const commissionShare = stakerPot * commission * (backedBy / totalStake);

  // share of incentive pot via piecewise sqrt, against sum of all validators' weights
  const myWeight = incentiveWeight(ownStake);
  const totalWeight = totalValidators * incentiveWeight(meanSelfStake); // approximation
  const incentiveShare = incentivePot * (myWeight / totalWeight);

  return { dot: ownShare + commissionShare + incentiveShare, pusd_per_year: pusdEnabled ? 24000 : 0 };
}
```

---

## Design notes

- **Single HTML file**, no build, open with `python3 -m http.server` or double-click.
- **Chart.js + plain SVG** (for flow diagram). Total weight under 200KB.
- **No persistence** — refresh = defaults. URL params for presets (`?preset=launch`).
- **Sliders everywhere** (per feedback). Range/step chosen so single drags feel meaningful.
- **Show formulas in collapsible "Math" panel below charts** for the curious.
- **"Source" tooltip** on each input — links to the Rust file/line where the storage item lives.

---

## What we deliberately dropped (vs v1)

- ❌ Sliders for issuance constants (`IssuanceCadence`, `MaxElapsedPerDrip`, `HARD_CAP_TARGET`, `BI_ANNUAL_RATE`, etc.) — fixed, not OpenGov-tunable, irrelevant to the audience.
- ❌ Self-stake curve sliders (T/C/k) — too researcher-y. Curve uses spec defaults internally.
- ❌ Cost-of-attack KPI — out of scope for this audience.
- ❌ POSS multiplier — too speculative.
- ❌ Avg validator commission slider, leak slider — leak surfaces as a KPI, commission is on the probe validator only.
- ❌ Dedicated Chart 3 (self-stake curve plot) — folded into the math panel for the curious.

---

## Open questions before I build

1. **Year-vs-time-series:** the year slider drives KPIs and the validator-income chart. Should Chart 1 also show *just* the selected year (vertical line + readout) or the full 10-year sweep with the year as a marker? I drafted "full sweep with marker" — confirm. 
> Lets try with your suggestion. Not sure yet.
2. **Probe validator vs typical validator:** Group 3 has a "probe validator" the user configures, plus Chart 2 also shows "typical nominator per 1,000 DOT". Two personas. OK or too much?
> This is fine.
3. **pUSD amount:** I hardcoded $2k/month. Should this be a slider, or stay fixed?
> Slider
4. **Presets — illustrative split %s:** the "Self-stake enabled" preset uses 14/80/6, "Q4" uses 12/70/6/12. Pure guesses. If you have actual planned numbers, paste them and I'll lock them in.
> Default to 15/85/0 and let users choose/change. Just one preset.
5. **Total stake / total validators:** for the income formula I need a baseline — current chain values (~600 validators, ~700M DOT staked). Confirm or override.
> Total validator: 600, minimum stake: 1,276,095 DOT, sum stake: 885,012,465
