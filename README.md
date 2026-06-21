# CausalTrace Lab v0.7 — Verified Calibration

A local-first, browser-based benchmark for testing a narrow question:

> Can a policy improve safely on hidden transition-mechanic shifts after it collects evidence, verifies that evidence out-of-sample, and only then permits limited action overrides?

CausalTrace Lab is an interactive single-file experiment. It compares a public-factor Q-learning baseline with a verification-gated adaptive policy in a controlled gridworld containing portals, gravity shifts, echo timing, hazards, layout changes, and action noise.

## What v0.7 demonstrates

Under the default v0.7 protocol, **VerifiedGuard** outperformed **Composer** on the preregistered primary family: held-out gravity-direction and echo-cadence shifts.

| Primary result | Default export |
|---|---:|
| Mean paired return difference | **+2.409** |
| Bootstrap 95% confidence interval | **[+0.730, +3.986]** |
| Median seed difference | **+2.928** |
| Positive independent seeds | **6 / 10** |
| Frozen verification accuracy | **100%** |
| Verified deployment rate | **100%** |
| Layout and noise guard rails | **Passed** |

This supports a narrow conclusion for this toy environment:

> A verified, evidence-gated local adaptation layer improved performance on hidden direction/cadence shifts without failing the tested layout or action-noise safety checks.

It does **not** demonstrate general causal intelligence, causal discovery in open environments, or real-world reliability.

---

## Why this benchmark exists

Earlier versions showed why average gains alone are not enough:

- **v0.4**: compositional representations improved some held-out conditions, but not direction/cadence shifts.
- **v0.5**: a learned dynamics layer had an inconclusive primary result and regressed on layout recombination.
- **v0.6**: an evidence-gated adaptive policy produced a positive mean but improved only 4 of 10 seeds.
- **v0.7**: separates discovery from verification, requires seed-level replication, checks the median effect, records whether the mechanism was actually deployed, and preserves non-regression guard rails.

The aim is not to produce a flattering score. The aim is to make a claim that can fail visibly.

---

## Agents compared

### Composer

A Q-learning policy using only public state and public task factors.

- Uses additive, factorised Q-values.
- Does **not** receive hidden gravity direction or hidden echo cadence in its policy state or factor keys.
- Cannot collect or verify episode-local mechanism evidence.

### VerifiedGuard

The same public Q-learning base policy, plus a limited local adaptation layer.

1. **Discovery packet** — receives fixed probe actions and records observed transitions.
2. **Frozen posterior** — converts discovery evidence into beliefs over hidden direction/cadence mechanisms.
3. **Verification packet** — makes held-out predictions using the frozen posterior. Verification does not update the posterior.
4. **Deployment gate** — permits local overrides only when verification, confidence, and a safety/utility test all pass.
5. **Abstention** — defaults to the Composer action whenever evidence is weak or an override is not certified.

This design makes the adaptation mechanism inspectable and falsifiable.

---

## Benchmark design

### Environment

The gridworld includes:

- A start position and fixed goal
- Hazard cells
- Portal/gate mechanics
- Gravity that changes motion direction
- Echo events whose timing depends on a hidden cadence
- Alternate layouts
- Optional action-slip noise

The hidden variables under test are:

- **Gravity direction**
- **Echo cadence**

They are deliberately excluded from both agents’ policy state and public factor keys.

### Held-out evaluation families

| Family | Purpose | Role |
|---|---|---|
| Clean recombination | New combinations of known mechanics | Secondary evidence |
| Layout recombination | Mechanisms in altered geometry | Safety guard rail |
| Direction & cadence | Hidden transition-rule shifts | **Primary test** |
| Action-noise stress | Mechanisms under action-slip noise | Safety guard rail |

### Default protocol

| Setting | Value |
|---|---:|
| Master seed | `24601` |
| Paired independent seeds | `10` |
| Training episodes per seed | `1000` |
| Discovery probes | `8` |
| Frozen verification probes | `4` |
| Evaluation repeats per case | `5` |
| Learning rate | `0.22` |
| Discount factor | `0.94` |
| Exploration rate | `0.22` |
| Action-slip noise | `0.12` |

---

## Preregistered decision rule

The v0.7 primary claim is supported only when **all** of the following are true:

1. The paired bootstrap 95% confidence interval lower bound for `VerifiedGuard − Composer` return is greater than zero.
2. At least 60% of independent seeds have a positive primary-family difference.
3. The median seed-level primary difference is greater than zero.
4. Primary held-out verification accuracy is at least 90%.
5. Verified deployment occurs in at least 20% of primary episodes.
6. The lower confidence bounds for layout and action-noise guard rails are each at least `−0.25`.

A positive average alone is not sufficient.

---

## Default v0.7 result

The supplied default export passed the v0.7 decision rule.

| Held-out family | Composer return | VerifiedGuard return | Difference | 95% CI | Positive seeds |
|---|---:|---:|---:|---:|---:|
| Clean recombination | -1.405 | 4.524 | +5.929 | [+5.786, +6.073] | 10 / 10 |
| Layout recombination | -7.907 | 4.041 | +11.949 | [+11.409, +12.338] | 10 / 10 |
| **Direction & cadence** | **-4.573** | **-2.164** | **+2.409** | **[+0.820, +4.034]** | **6 / 10** |
| Action-noise stress | -4.185 | -2.667 | +1.518 | [+0.720, +2.423] | 7 / 10 |

The action-noise group improved in return, but its mechanism-prediction accuracy was lower than the primary family. It is therefore evidence of non-regression under the defined guard rail, not proof of equally strong hidden-mechanic identification under noise.

---

## Run the benchmark

No build step or package installation is required.

1. Open `causaltrace_lab_v07.html` in a modern desktop browser.
2. Keep the default controls for the reference protocol, or select new values deliberately.
3. Select **Run full benchmark**.
4. Review the result card, confidence interval, seed ledger, group table, evidence-gate audit, and policy replays.
5. Select **Export JSON** to save the complete configuration, per-seed outcomes, calibration audit, and result summary.

### Interface controls

- **Run full benchmark** — completes the paired multi-seed experiment.
- **Run 120 episodes / seed** — advances the current experiment in visible chunks.
- **Reset** — clears the in-memory run.
- **Export JSON** — saves the reproducibility bundle.
- **Policy replay** — animates Composer or VerifiedGuard through a chosen held-out case and seed.

Browser settings are retained locally for convenience. Exported JSON is the authoritative record of a completed run.

---

## Reading the output

### A supported result means

The specified adaptive method beat the specified baseline in this specific toy benchmark, under the declared seed count, probe budget, world distribution, metrics, and guard rails.

### A failed result means

The benchmark intentionally preserves a failure verdict when any criterion is missed. Examples include:

- An interval that crosses zero
- Too few positive seeds
- A non-positive median effect
- Poor frozen-packet prediction accuracy
- Too little actual deployment
- A harmful layout or noise regression

A failed result is useful evidence. It identifies where the proposed mechanism is not reliable enough.

---

## Limitations

This project has strict limits:

- It is a small, hand-designed gridworld.
- The hypothesis class and mechanics are intentionally simple and finite.
- Probe actions and hidden mechanism families are designed by the experimenter.
- Bootstrap intervals quantify uncertainty over the configured seeds, not universal generalisation.
- Results do not establish causal discovery, human-level reasoning, broad planning ability, or real-world safety.
- Independent replication should use unseen maps, different transition rules, larger seed counts, and an implementation that is not tuned against the same result set.

Treat the current result as a **reproducible prototype finding**, not a general AI claim.

---

## Suggested next experiments

1. Run 30–100 paired seeds with a locked configuration and independent test seeds.
2. Randomise starts, goals, hazards, and gate positions procedurally.
3. Hold out entirely new gravity functions and non-binary echo periods.
4. Charge an explicit reward cost for every calibration action.
5. Add a matched model-based RL baseline and a recurrent policy baseline.
6. Run ablations: remove verification, remove the deployment gate, remove the safety certificate, and vary probe budgets.
7. Use an externally generated test suite to reduce evaluation leakage.
8. Publish the fixed benchmark specification and raw exports before tuning further.

---

## Repository contents

```text
causaltrace_lab_v07.html                 Interactive local benchmark
causaltrace-v07-verified-calibration-24601.json
                                         Reference export for master seed 24601
README.md                                Project documentation
```

---

## Claim boundary

**Supported in this toy protocol:**

> With fixed discovery probes, a separate frozen verification packet, and confidence/safety-gated action overrides, VerifiedGuard improved held-out hidden direction/cadence performance over Composer while passing defined layout and action-noise guard rails.

**Not supported:**

> General causal intelligence, real-world robustness, universal transfer, or a claim that the method will work outside this controlled benchmark.

---

## License

Add the licence you intend to use before public release. For experimental code, an explicit license is better than leaving reuse terms unclear.
