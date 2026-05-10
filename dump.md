# DAP / Staking-Async — External Source Dump
_Pulled 2026-05-10 for retreat prep_

## 1. Spec — HackMD (jonasW3F/rkN6BXE2ex)
**URL:** https://hackmd.io/@jonasW3F/rkN6BXE2ex

**Key points:**
- Single algorithmic budget (DAP) replaces hard-coded staking issuance; protocol revenue (coretime, tx fees) flows in instead of being burned.
- Validator self-stake rewarded via piecewise sqrt curve with diminishing returns above `T`, flat above `C`.
- Nominators: slashing removed entirely; unbonding ≤ 1 era (~1 day).
- ELVES protocol used to derive validator security floor.
- Strategic Reserve sized via dynamic-programming consumption smoothing.

**Parameters / formulas:**
- `OptimumSelfStake` (T) = 30,000 DOT
- `HardCap` (C) = 100,000 DOT
- `SlopeFactor` (k) = 0.5
- Min self-stake = 10,000 DOT
- Validator count = 600
- Target APY at T = 30%
- Operational payment = $2,000 per validator per **year** (HackMD wording — note: forum proposal says per **month**; see Conflicts)
- DOT price assumption = $3
- Stablecoin overcollateralization (CR) = 150%
- ELVES security parameter ε = 1/20001; constraint `1/ε > N/ν + 1`
- Required validator stake fraction: > 3% of DOT market cap (~90k DOT/validator)
- Total supply cap = 2.1B DOT
- Strategic Reserve: horizon 30y, discount β = 0.91, year-1 consumption C₀ = 40,056,081 DOT, year-1 reserve allocation = 15,505,081 DOT
- Issuance reduction at transition = 53.6%; post-transition annual inflow = 55,561,162 DOT
- Year-1 budget split: validators-resilience 5.4M, validators-ops (stable) 7.2M, treasury 8,352,158, nominators 19,103,923, reserve 15,505,081 DOT
- Validator weight: `w(s) = √s` for s ≤ T; `√(T + k²(s−T))` for T < s ≤ C; `√(T + k²(C−T))` above C
- Daily payout: `π_i = B_day · w(s_i) / Σ w(s_j)`
- Consumption path: `C_t = C_0 · β^t` (β=0.91)

**Open questions / decisions pending:**
- DOT/stablecoin split for treasury beneficiary payments — committee-decided, not fixed.
- Governance mechanism (committee vs OpenGov w/ advisory board) — leans OpenGov, deferred.
- Stablecoin implementation (vault, liquidations) — out of scope.
- Whether logic lives in one pallet vs distributed — engineering decision.
- Spec carries explicit "nothing here is final" caveat.

---

## 2. DAP Forum Proposal (#15878)
**URL:** https://forum.polkadot.network/t/proposal-dynamic-allocation-pool-dap/15878

**Key points:**
- Same overall numbers as HackMD; published 2025-11-10.
- Targeted launch window: March 2026 alongside new issuance curve.
- Community feedback adjusted some defaults; others still under discussion.

**Parameters / formulas:**
- T = 30k, C = 100k, k = 0.5 (provisional)
- Min self-stake = 10k DOT (locked-in)
- Operational payment = **$2,000/month** per validator (forum text — conflicts with HackMD's per-year)
- $14.4M annual ops cost ÷ $3/DOT × 1.5 CR = 7.2M DOT (validates per-month reading)
- Year-1 inflow = 55,561,162 DOT for 2 years thereafter
- Treasury split (proposed): 50/50 stables/DOT — under discussion
- ELVES, β=0.91, T=30y, all identical to spec.

**Open questions / decisions pending:**
- $2k/month flagged as too low by SAXEMBERG ("we went from 5k to 2k") — Jonas: needs more discussion.
- Governance: ChrawnnaCorp pushed time-limited committee with 14/28-day reversion; Jonas prefers whitelist+veto.
- joepetrowski floated minting full remaining supply to cap directly into DAP as "Issuance Buffer" — not adopted, debated.
- Whitelisted-account definition for self-stake placement — TBD.

**Timeline:**
- 2025-11-10: proposal posted.
- March 2026: launch window.
- Q2 2026: nominator changes (per kianenigma).

---

## 3. Referendum 1827 — DAP Phase 1
**URL:** https://polkadot.subsquare.io/referenda/1827

**Status:** Passed & executed. Track: **Wish For Change**. Enactment on/before **2026-03-14**.

**On-chain decision:**
- Stand up basic DAP pallet with permanent holding account.
- Stop burning fees / coretime / treasury — redirect to DAP.
- Slashed DOT → DAP (not burned).
- Approve direction for staking parameter changes (validator min-stake, min-commission, nominator unslashable + fast unbond).

**Parameters approved:**
- Validator min self-stake: 10,000 DOT
- Validator min commission: 10%
- Nominator slash protection (exempt)
- Nominator unbonding: 24–48 h (replaces 28d)

**Vote tally:**
- Aye: ~14.71M DOT (≈100%)
- Nay: ~10.3 DOT (rounding-dust)
- Support: 0.29% (~4.77M DOT) vs 50% threshold — passed via WFC track curve, not raw support.
- Decision period 28d, confirmation 1d.

---

## 4. Forum: March 2026 Changes (#17101)
**URL:** https://forum.polkadot.network/t/changes-on-polkadot-in-march-2026/17101

**Key dates (absolute):**
- 2025-09-14 — supply cap approved by governance (Ref 1710).
- **2026-03-14 (Pi Day)** — first issuance reduction takes effect.
- End of March 2026 — Runtime 2.1.1 live.
- **2026-03-23** — Runtime 2.1.1 confirmed live; StakingOperator proxy available.
- End of April 2026 — governance proposal expected for validator min requirements.
- Q2 2026 — nominator parameter changes.

**Parameters / formulas:**
- Hard cap: 2.1B DOT.
- Pre-transition annual issuance: 120M DOT.
- Post-transition annual issuance: **55M DOT** (this source rounds; HackMD/forum: 55,561,162; timeline post: 55.8M — minor disagreement).
- Halving rule: every 2 years reduce by **13.14% of remaining supply** (not a clean halving — explicit %).
- Phase-1 staking: 10k min self-stake (slashable), 10% min commission.
- Asset Hub session-key deposit: ~60 DOT (Polkadot), ~3 KSM (Kusama).
- Phase-2 (pending Q2–Q3 2026): T=30k, C=100k, APY@30k = 30%, **APY@100k = 9%**, ops $2k/month/node in stables.
- Nominator: 28d → 24–48h unbonding; unslashable.

**Runtime 2.1.1 contents:**
- Basic DAP pallet, treasury burns → DAP, validator slashes → DAP, StakingOperator proxy, AH session-key mgmt.

**References:** Ref 1710 (cap), Ref 1827 (DAP P1), Kusama Ref 637 (2.1.0).

**Open:** Phase-2 details "may change" pending OpenGov ratification.

---

## 5. Forum: Staking Progress Timeline (#17436)
**URL:** https://forum.polkadot.network/t/polkadot-staking-changes-progress-timeline/17436

**Past milestones (now confirmed):**
- 2026-03-14: issuance 120M → ~55.8M; cap 2.1B w/ decreasing trajectory every 2y.
- 2026-03-23: Runtime 2.1.1; DAP collects slashes; treasury burns halted.
- 2026-04-01: Min validator commission 10% enforced via **Ref 1872** (temporary, until budget split).

**Projected milestones:**
- End of May 2026: 10k min self-stake enforced (chilling for non-compliant); nominators unslashable + 2-day unbond active.
- Mid-June 2026: budget split implemented; **~70% APR on 30k DOT self-stake**; commission system removed.
- End of 2026: stablecoin payments for ops; 1-year vesting on self-stake DOT incentives; dynamic validator-set sizing tied to coretime.

**Initial-budget parameters:**
- Validator self-stake target: **70% APR on 30k DOT** = 21,000 DOT/validator/yr ⇒ 600 × 21k = **12.6M DOT/yr** total.
- Staker payouts: target **3% APR at 50% staking rate** on ~1.68B DOT supply ⇒ 25.2M DOT/yr.
- Reserve buffer: 18.0M DOT/yr retained from 55.8M issuance.
- Validator-set reduction: 600 (on 120 cores) → **250–300 validators on ~64 cores**, gated on RFC17 (Q3 2026 target).

**Decisions made (✓):** 10% min commission; 10k min self-stake; 2-day nominator unbond; nominator unslashable; 3% APR @ 50% stake.

**Pending (⧗):** stable-payment structure/timing; fixed ops $ amount; final validator-set size mechanism; DAP validator selection scoring fn; decentralization safeguards for 250–300 set.

**Risks flagged:** no formal quantitative model behind 70/3 split; concentration risk at 250–300 validators; nominator exodus (rewards down 70–80% since cut); DAP scoring fn undefined; stablecoin/RFC17/audit slippage; large-DAO referendum dominance.

**References:** Proposal #15878, Roadmap #16511, Ref 1827, RFC17, jonasw3f.github.io/economics_overhaul_hosted.

---

## Cross-cutting summary

**Confirmed numbers (≥2 sources):**
- Total supply cap: **2.1B DOT** (all sources).
- Issuance cut date: **2026-03-14** (#17101, #17436, internal staking.md via Ref 1710).
- Min self-stake: **10,000 DOT** (HackMD, forum, Ref 1827, #17101, #17436).
- Min commission: **10%** (Ref 1827, #17101, #17436).
- Nominator unbonding: 24–48h / 2-day; unslashable (Ref 1827, #17101, #17436).
- T = 30k, C = 100k (HackMD, forum, #17101).
- 600 validators today, target reduction (HackMD, #17436, staking.md).
- Operational payment denominated in **stablecoins** (HackMD, forum, #17101, #17436).
- Strategic Reserve / "buffer" exists (HackMD, forum, #17436 calls it "reserve buffer").
- DAP receives slashes + burns + fees (Ref 1827, #17101, #17436, staking.md).

**Conflicts between sources:**
- **Operational payment magnitude:** HackMD says "$2,000 per validator per year"; forum proposal #15878 + #17101 say "$2,000 per month". Per-month is internally consistent ($14.4M / yr = 600 × $2k × 12). HackMD wording almost certainly a typo; treat **$2k/month** as canonical.
- **Annual issuance post-transition:** HackMD/forum 55,561,162 DOT; #17101 rounds to 55M; #17436 says 55.8M. ~0.4M spread.
- **Validator self-stake APY target:** HackMD/forum say **30% APY at T=30k**; #17436 says **70% APR on 30k DOT** in initial-budget plan ⇒ 21k DOT/validator/year. These are not the same number — 70% appears in the post-budget-split rollout, suggesting target moved up between spec and current rollout plan, OR #17436 is describing pre-curve flat allocation before T/C/k take effect.
- **Halving phrasing:** staking.md says "halving every 2 years"; #17101 says "reduce by 13.14% of remaining supply every 2 years". 13.14% ≠ 50% — these aren't the same operation. The "halving" label in staking.md is loose shorthand.
- **Staker APR target:** #17436 cites 3% APR at 50% staking rate (25.2M DOT/yr). HackMD/forum year-1 nominator budget = 19.1M DOT — distinct number, distinct framing.
- **DAP launch wording:** Ref 1827 + #17101 ship DAP P1 in **March 2026**; staking.md says "release 2.1.0 — Feb 2026". 2.1.0 vs 2.1.1 confusion — runtime 2.1.1 is what actually went live (March, not Feb).
- **Validator-set size endpoint:** #17436 says 250–300; staking.md says "e.g. 600 → 300". Range vs midpoint.
- **`MaxValidatorBond` / hard-cap-above-curve language:** HackMD calls it `HardCap (C)`, forum calls it "self-stake cap"; staking.md just says "cap C". No conflict, just different labels.

**What's not in any source (gaps the simulator must handwave):**
- Exact **per-block drip** rate / formula (continuous mint into DAP buffer) — only annual totals given; sim needs to assume `block_inflation = annual_inflow / blocks_per_year`.
- **Budget split percentages** between DAP outflows (staker rewards / self-stake / treasury / reserve / pUSD-ops) — HackMD gives year-1 DOT totals but no governance-tunable %; staking.md explicitly says "what is NOT fixed: how new issuance is split".
- **DAP validator selection / scoring function** for the future smaller set — flagged as undefined by #17436.
- **pUSD acquisition mechanics** — staked-DOT vault flow named but parameters (target CR per regime, healing mechanism) absent.
- **POSS reward multiplier `m`** value and PoP-verification weight cap — staking.md mentions concept; no source quantifies.
- **Slashing destination accounting** post-DAP — funds "deactivated" in buffer, but reactivation rules unspecified.
- **Vesting schedule details** for self-stake DOT (1-year vesting mentioned but no cliff / linearity).
- **Dust/burn aggregation thresholds** on satellite chains and XCM batching cadence.
- **Halving curve precise form** between the 13.14% step and the asymptote toward 2.1B — interpolation rule not given.
