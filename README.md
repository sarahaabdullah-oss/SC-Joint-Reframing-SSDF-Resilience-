# SC-Joint-Reframing-SSDF-Resilience-
# SC-Joint — Security-Constrained Cross-Layer Defence against SSDF in Cognitive Radio Networks

Reproducibility package for the manuscript *"SC-Joint: reframing SSDF resilience
as a network-design problem in cooperative cognitive radio networks."*

SC-Joint reframes Spectrum Sensing Data Falsification (SSDF) resilience from an
attack-detection problem into a **network-layer resource-optimization** problem:
malicious secondary users (SUs) are excluded by an oracle-free trust layer before
fusion, and the surviving honest set is re-optimized over the cooperating-set size
K and sensing time τ under SLA constraints on detection, throughput, and lifetime.

---

## 1. Requirements

- **MATLAB** R2019b or later (primary), **or GNU Octave** 6+ (headless cross-validation).
- No toolboxes required — statistics helpers (`tiedrank`, CIs) are reimplemented locally.
- Plotting scripts use Python 3 with `matplotlib`, `numpy` (optional, figures only).

## 2. One-command reproduction

```matlab
% from the repository root, in MATLAB or Octave:
run_all            % executes every scenario + the three additional analyses
```

Or run any scenario individually (see Section 4). Each script prints its results
table to the console and writes a CSV with the same numbers.

## 3. Fixed seeds and paired design

All experiments use fixed seeds (`rng(seed)` per trial). Multi-condition comparisons
use **paired common random numbers**: every scheme is evaluated on the same
environment realization per Monte-Carlo trial, and attackers are injected by
**cumulative nesting** on a fixed base topology. This isolates the effect of adding
attackers from between-topology variation and avoids artifactually non-monotone
curves. Parameters are centralized in each script's `params()` and match **Table II**
of the manuscript.

## 4. Scenarios (map to the paper)

| Script | Scenario | Produces | Attack model |
|--------|----------|----------|--------------|
| `scenario1_tableII.m` | Sc 1 | Attack-free cross-layer baseline; feasibility 100%, R ≈ 2.15 | — |
| `scenario3_tableII.m` | Sc 3 | Beta–Bernoulli reputation (LOO); honest 0.80 / attacker 0.12, 100%/100% split (Fig. 7) | always-yes |
| `scenario4_tableII.m` | Sc 4 | Feasibility vs attacker fraction β, independent draws (Fig. 8) | always-yes |
| `scenario5_tableII.m` | Sc 5 | Feasibility vs attacker count, paired CRN → **Table III / Fig. 9**; 98→86% | always-yes |
| `scenario6_tableII.m` | Sc 6 | Cooperative ROC, 2 attackers → **Fig. 10**; AUC 0.96/0.87/0.50 | always-yes + flip |
| `scenario7_tableII.m` | Sc 7 | AUC vs attacker count → **Fig. 11 / Table IV**; SC-Joint 0.97→0.94 | flip p=0.5 |
| `scenario8_network22.m` | Sc 8 | Network stress-test → **Fig. 12**; SC 100→95%, Outlier 66.7→76.7% | flip p=0.9 |
| `scenario9_tableII.m` | Sc 9 | Stealth sweep → **Fig. 13**; detection valley AUC≈0.65 @ p≈0.3 | flip, p swept |

Scenario 2 (always-yes throughput impact) is illustrative and folded into Sc 1/Sc 5.

## 5. Additional analyses (revision)

| Script | Produces | Purpose |
|--------|----------|---------|
| `scenario5_thm2.m` | **Theorem-2 validation figure**; 0/300 violations, 0 attackers admitted, \|S\|=N−k | Direct test of the monotonicity theorem on the paired-CRN run (same numbers as Table III by construction) |
| `ablation_tableVI.m` | **Table V** (ablation); Beta-only 100% = SC-Joint, Optimizer-only 85% | Isolates each component; shows the trust layer is decisive |
| `coalition_test.m` | **Table VI** (admission FP/FN); colluding/on-off/adaptive vs β | Reputation robustness under coordinated & adaptive attacks |
| `adaptive_impact.m` | Adaptive-mimic harm (ΔP_d, Δfeasibility vs flip fraction) | Shows the admitted adaptive mimic is non-damaging (ΔP_d ≤ 0.006) |

Master / library:
- `CrossLayerCRN_TableII.m` — full master run (Table II parameters: r_th=0.55, κ=3,
  300 topologies) reproducing Table IV within ±0.003.

## 6. The Outlier/MAD baseline

The baseline is the consensus-deviation outlier rejection of **Kaligineedi,
Khabbazian & Bhargava, IEEE Trans. Wireless Commun., 9(8):2488–2497, 2010**, adapted
to run without ground-truth labels. It operates on each SU's **report rate**
r_i = (1/B) Σ_t d_i(t) over a window of B sensing periods; with m = median(r) and
MAD = median(|r − m|), SU i is retained iff **|r_i − m| ≤ κ·MAD** (two-sided; κ per
Table II). Survivors are OR-fused. Crucially, the baseline is granted the **same
cross-layer re-optimization** (τ, K) as SC-Joint — the only difference is the
admission rule — so the comparison is symmetric. Its low attack-free feasibility
(66.7%) is intrinsic: the two-sided band trims honest detectors at both extremes of
the dispersed report-rate distribution.

## 7. Key results (source of every number)

| Quantity | Value | Source |
|----------|-------|--------|
| Feasibility, no attack | 100% | Sc 1 |
| Feasibility, k=0→4 (paired) | 98 → 86% | Sc 5 / Table III |
| Feasibility, stress-test | 100 → 95% | Sc 8 |
| Theorem-2 monotonicity | 0/300 violations, 0 attackers admitted | scenario5_thm2 |
| Throughput | 1.69 – 2.16 Mb/s | Sc 1–9 (valley = Sc 9) |
| Detection AUC (always-yes, 2 att.) | 0.96 / 0.87 / 0.50 | Sc 6 |
| Detection AUC, k=0→4 (flip) | 0.97 → 0.94 | Sc 7 / Table IV |
| Stealth valley | AUC ≈ 0.65 @ p≈0.3 | Sc 9 |
| Ablation: reputation alone | 100% (= SC-Joint) | Table V |
| Reputation vs colluding (β=0.25) | 57% admitted | Table VI |
| Adaptive mimic harm | ΔP_d ≤ 0.006 (non-damaging) | adaptive_impact |
| Global feasibility envelope | 80 – 100% | Abstract |

Values from different scenarios are **scenario-specific**, not pooled: each fixes its
own attack model, β, and p. Table VII in the manuscript maps every reported value to
its source experiment.

## 8. Scope and limitations

The trust layer is oracle-free (no PU ground-truth labels) and resists
**non-coordinated, below-majority** SSDF. Coordinated collusion and majority-forming
attacks lie outside the guaranteed envelope; a consensus-mimicking adaptive attacker
is admitted but shown to be non-damaging. "Oracle-free" refers to operation without
ground-truth labels, not universal identifiability.

## 9. Files

```
scenario1_tableII.m         Sc 1        scenario5_thm2.m           Theorem-2 validation
scenario3_tableII.m         Sc 3        ablation_tableVI.m         Table V (ablation)
scenario4_tableII.m         Sc 4        coalition_test.m           Table VI (robustness)
scenario5_tableII.m         Sc 5        adaptive_impact.m          adaptive-mimic harm
scenario6_tableII.m         Sc 6        CrossLayerCRN_TableII.m    master run
scenario7_tableII.m         Sc 7        *.csv                      raw results per script
scenario8_network22.m       Sc 8
scenario9_tableII.m         Sc 9
```

## 10. Citation

If you use this code, please cite the SC-Joint manuscript (see the paper for the
current reference) and the baseline of Kaligineedi et al. (2010).

## License

Released for academic reproducibility. See `LICENSE`.
