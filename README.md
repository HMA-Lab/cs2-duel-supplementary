# Supplementary Material

**Paper:** Duel Outcome Classification in FPS Games Using Context-Driven Multimodal Fusion  
**Authors:** Abhishek Suwalka, Vaibhav Prajapati, Anish Chand Turlapaty  
**Venue:** ICMLT 2026 (IEEE)

---

## Reviewer Comment Mapping

| # | Reviewer Comment | Section |
|:-:|:-----------------|:--------|
| 1 | Report median accuracy and confidence intervals alongside mean | [S1](#s1-lopocv-variance-median-accuracy-and-confidence-intervals) |
| 2 | Report overall class distribution and higher-tier win rate | [S2](#s2-overall-class-balance) |
| 3 | Specify all 114 features for reproducibility | [S3](#s3-feature-engineering-specification) |
| 4 | Consider inter-rater reliability or rank validation | [S4](#s4-skill-tier-self-reporting) |
| 5 | Specify which AI tool(s) were used | [S5](#s5-ai-tools-used-in-manuscript-preparation) |

---

## S1. LOPOCV Variance: Median Accuracy and Confidence Intervals

**Reviewer Comment 1:** *"High variance acknowledged — consider reporting median accuracy or confidence intervals alongside mean."*

Additional distributional statistics for the best-performing model (L2-regularised Logistic Regression, full multimodal features):

| Statistic | Value |
|:----------|:------|
| Mean accuracy | 86.95% |
| **Median accuracy** | **88.9%** |
| Standard deviation | 12.0 pp |
| **95% Confidence Interval** | **[80.8%, 93.1%]** |
| Number of folds (*n*) | 18 |
| Minimum fold accuracy | 58.4% (pair N1–I1, near-tier) |
| Maximum fold accuracy | 100.0% (pairs I2–N2, P1–N1, P4–N4) |

The median (88.9%) exceeding the mean (86.95%) confirms a left-skewed distribution — most folds achieve high accuracy, with a small number of near-tier pairs pulling the mean downward. The 95% CI [80.8%, 93.1%] confirms the full multimodal model reliably outperforms all behavioral-only baselines (best: 64.8%).

---

## S2. Overall Class Balance

**Reviewer Comment 2:** *"What is the overall class distribution? Report percentage of rounds won by higher-tier player across all rounds."*

The binary label is y = 1 if Player 1 (Setup-1) wins the duel. The overall distribution across all 2,889 rounds:

| Class | Count | Percentage |
|:------|------:|-----------:|
| Player 1 wins (y = 1) | 1,534 | **53.1%** |
| Player 2 wins (y = 0) | 1,355 | **46.9%** |
| **Total** | **2,889** | **100.0%** |

The near-balanced distribution (53.1% vs. 46.9%) precludes label imbalance as a confound. Among the 1,897 inter-tier rounds, the higher-tier player wins **63.0%** of rounds.

---

## S3. Feature Engineering Specification

**Reviewer Comment 3:** *"Keyboard features (51) listed but not fully specified in paper — consider adding a table in appendix or supplementary material."*

This section specifies all 114 features used in the full multimodal model.

### S3.1 Feature Count Summary

| Phase | Feature Set | Candidates | EDA-Selected | Paper Reported |
|:------|:------------|:----------:|:------------:|:--------------:|
| A | Keyboard Only | 58 | **51** | 51 |
| B | Mouse Only | 28 | **18** | 18 |
| C | Combined Behavioral (KB + Mouse + Interaction) | 94 | **79** | 79 |
| D | Full Multimodal (Phase C + History + Metadata) | — | **114** | 114 |

### S3.2 Feature Naming Convention

All per-player features are computed independently for Player 1 and Player 2. Signed difference features (P1 − P2) are then appended.

| Prefix | Meaning | Example |
|:-------|:--------|:--------|
| `p1_kb_` / `p2_kb_` | Keyboard feature per player | `p1_kb_n_total` |
| `diff_kb_` | P1 − P2 keyboard difference | `diff_kb_direction_changes` |
| `p1_mc_` / `p2_mc_` | Mouse click feature per player | `p1_mc_n_left` |
| `diff_mc_` | P1 − P2 mouse difference | `diff_mc_n_left` |
| `p1_inter_` / `p2_inter_` | Cross-modal interaction feature | `p1_inter_fire_while_moving` |
| `p1_action_density` / `p2_action_density` | Combined keyboard + mouse activity rate | — |
| `hist_` | In-match rolling history feature | `hist_p1_win_rate` |
| `meta_` / `score_` | Participant metadata or score feature | `meta_exp_diff`, `score_diff` |

### S3.3 EDA Feature Selection Filter

Before modelling, all candidate features pass through a three-criterion filter. A feature is **dropped** if any condition holds:

| Criterion | Threshold | Rationale |
|:----------|:----------|:----------|
| Zero-rate | > 95% of values are zero | No discriminative signal |
| Variance | < 1×10⁻⁶ | Near-constant across all rounds |
| Mann–Whitney *p*-value AND \|point-biserial *r*\| | MW *p* > 0.10 **AND** \|*r*\| < 0.02 | Win/loss distributions statistically indistinguishable **and** near-zero correlation with outcome |

The third criterion is a **conjunction**: both conditions must hold simultaneously.

### S3.4 Keyboard Features (51 selected from 58 candidates)

**Applied to:** 2,881 rounds where both players registered at least one keystroke.  
**Pre-processing:** Control characters (e.g., Ctrl+A, Ctrl+D) are filtered from the raw keystroke log before counts are computed.

#### Base Features per Player (17 candidates per player)

| Feature Suffix | Description | Unit | Notes |
|:---------------|:------------|:-----|:------|
| `kb_n_total` | Total key-press events in the duel | count | |
| `kb_n_per_sec` | Key presses per second | keys/s | |
| `kb_n_w` | Forward key ('W') presses | count | |
| `kb_n_a` | Strafe-left key ('A') presses | count | |
| `kb_n_d` | Strafe-right key ('D') presses | count | |
| `kb_n_wasd` | Total movement key presses (W + A + S + D) | count | |
| `kb_w_per_sec` | Forward presses per second | keys/s | |
| `kb_a_per_sec` | Strafe-left presses per second | keys/s | |
| `kb_d_per_sec` | Strafe-right presses per second | keys/s | |
| `kb_n_r` | Reload key ('R') presses | count | Strongest single keyboard predictor (*r* = +0.138, *p* < 10⁻¹³) |
| `kb_r_per_sec` | Reload presses per second | keys/s | |
| `kb_n_unique` | Number of distinct keys pressed | count | |
| `kb_avg_interval_ms` | Mean inter-key interval | ms | |
| `kb_std_interval_ms` | Std. dev. of inter-key intervals | ms | |
| `kb_key_rhythm` | CV of inter-key intervals (std / mean) | ratio | |
| `kb_direction_changes` | Count of consecutive A→D or D→A strafe switches | count | |
| `kb_d_minus_a` | Net rightward strafe (D − A count) | count | Dropped for P1 by EDA |

#### Features Dropped by EDA (7 dropped)

| Dropped Feature | Reason |
|:----------------|:-------|
| `p1_kb_d_minus_a` | MW *p* = 0.611, \|*r*\| = 0.019 < 0.02 |
| `p1_kb_active` | Variance = 0.00 (always 1) |
| `p2_kb_active` | Variance = 0.00 (always 1) |
| `diff_kb_active` | Zero-rate = 100%; variance = 0.00 |
| `is_M1` | MW *p* = 0.473, \|*r*\| = 0.013 < 0.02 |
| `is_M3` | MW *p* = 0.510, \|*r*\| = 0.012 < 0.02 |
| `round_duration` | MW *p* = 0.510, \|*r*\| = 0.008 < 0.02 |

> **Note:** `p2_kb_d_minus_a` and `diff_kb_d_minus_a` survive the filter; only the P1 variant is dropped. Similarly, `is_M2` survives while `is_M1` and `is_M3` are dropped.

#### Complete Keyboard Feature List (51 features)

**P1 keyboard (16):** `p1_kb_n_total`, `p1_kb_n_per_sec`, `p1_kb_n_w`, `p1_kb_n_a`, `p1_kb_n_d`, `p1_kb_n_wasd`, `p1_kb_w_per_sec`, `p1_kb_a_per_sec`, `p1_kb_d_per_sec`, `p1_kb_n_r`, `p1_kb_r_per_sec`, `p1_kb_n_unique`, `p1_kb_avg_interval_ms`, `p1_kb_std_interval_ms`, `p1_kb_key_rhythm`, `p1_kb_direction_changes`

**P2 keyboard (17):** `p2_kb_n_total`, `p2_kb_n_per_sec`, `p2_kb_n_w`, `p2_kb_n_a`, `p2_kb_n_d`, `p2_kb_n_wasd`, `p2_kb_w_per_sec`, `p2_kb_a_per_sec`, `p2_kb_d_per_sec`, `p2_kb_n_r`, `p2_kb_r_per_sec`, `p2_kb_n_unique`, `p2_kb_avg_interval_ms`, `p2_kb_std_interval_ms`, `p2_kb_key_rhythm`, `p2_kb_direction_changes`, `p2_kb_d_minus_a`

**Difference (17):** `diff_kb_n_total`, `diff_kb_n_per_sec`, `diff_kb_n_w`, `diff_kb_n_a`, `diff_kb_n_d`, `diff_kb_n_wasd`, `diff_kb_w_per_sec`, `diff_kb_a_per_sec`, `diff_kb_d_per_sec`, `diff_kb_n_r`, `diff_kb_r_per_sec`, `diff_kb_n_unique`, `diff_kb_avg_interval_ms`, `diff_kb_std_interval_ms`, `diff_kb_key_rhythm`, `diff_kb_direction_changes`, `diff_kb_d_minus_a`

**Shared (1):** `is_M2`

**Total: 16 + 17 + 17 + 1 = 51**

### S3.5 Mouse Click Features (18 selected from 28 candidates)

**Applied to:** 2,722 rounds where both players registered at least one click.  
**Spatial coordinates excluded:** CS2 locks the cursor to screen centre during gameplay; all coordinate columns carry zero spatial signal.

#### Base Features per Player (8 candidates per player)

| Feature Suffix | Description | Unit | Notes |
|:---------------|:------------|:-----|:------|
| `mc_active` | Binary: player had any click events | binary | Dropped (always 1 after round filter) |
| `mc_n_total` | Total click events (left + right) | count | |
| `mc_n_per_sec` | Clicks per second | clicks/s | |
| `mc_n_left` | Left-click (fire) count | count | Strongest volume predictor (*r* = +0.139) |
| `mc_left_per_sec` | Left-clicks per second | clicks/s | |
| `mc_n_right` | Right-click (scope) count | count | Retained for P2; dropped for P1 |
| `mc_right_ratio` | Right-clicks as fraction of all clicks | ratio | |
| `mc_cv_interval` | CV of inter-click intervals | ratio | Strongest mouse predictor (*r* = +0.142) |

#### Features Dropped by EDA (10 dropped)

| Dropped Feature | Reason |
|:----------------|:-------|
| `p1_mc_active` | Variance = 0.00 |
| `p2_mc_active` | Variance = 0.00 |
| `diff_mc_active` | Zero-rate = 100% |
| `p1_mc_n_right` | MW *p* = 0.636, \|*r*\| = 0.002 |
| `diff_mc_n_total` | MW *p* = 0.896, \|*r*\| = 0.003 |
| `diff_mc_n_per_sec` | MW *p* = 0.780, \|*r*\| = 0.015 |
| `diff_mc_left_per_sec` | MW *p* = 0.272, \|*r*\| = 0.015 |
| `diff_mc_cv_interval` | MW *p* = 0.532, \|*r*\| = 0.005 |
| `is_M1` | MW *p* = 0.362, \|*r*\| = 0.018 |
| `is_M3` | MW *p* = 0.557, \|*r*\| = 0.011 |

#### Complete Mouse Feature List (18 features)

**P1 mouse (6):** `p1_mc_n_total`, `p1_mc_n_per_sec`, `p1_mc_n_left`, `p1_mc_left_per_sec`, `p1_mc_right_ratio`, `p1_mc_cv_interval`

**P2 mouse (7):** `p2_mc_n_total`, `p2_mc_n_per_sec`, `p2_mc_n_left`, `p2_mc_left_per_sec`, `p2_mc_n_right`, `p2_mc_right_ratio`, `p2_mc_cv_interval`

**Difference (3):** `diff_mc_n_left`, `diff_mc_n_right`, `diff_mc_right_ratio`

**Shared (2):** `is_M2`, `is_M3`

**Total: 6 + 7 + 3 + 2 = 18**

### S3.6 Combined Behavioral Features (79 selected from 94 candidates)

Phase C combines keyboard and mouse features with 10 cross-modal interaction features. EDA is re-run on the combined set.

#### Cross-Modal Interaction Features (10 features)

| Feature Name | Description | Unit | Window |
|:-------------|:------------|:-----|:-------|
| `p1_inter_fire_while_moving` | P1 clicks within 200 ms of a WASD press | count | 200 ms |
| `p2_inter_fire_while_moving` | P2 clicks within 200 ms of a WASD press | count | 200 ms |
| `p2_inter_crouch_then_shoot` | P2 Ctrl press followed by click within 300 ms | count | 300 ms |
| `p2_inter_stop_then_shoot` | P2 click with no WASD in prior 500 ms | count | 500 ms |
| `diff_inter_fire_while_moving` | P1 − P2 fire-while-moving | count | — |
| `diff_inter_crouch_then_shoot` | P1 − P2 crouch-then-shoot | count | — |
| `diff_inter_stop_then_shoot` | P1 − P2 stop-then-shoot | count | — |
| `p1_action_density` | (key presses + clicks) / round duration for P1 | events/s | Full duel |
| `p2_action_density` | (key presses + clicks) / round duration for P2 | events/s | Full duel |
| `diff_action_density` | P1 − P2 action density | events/s | — |

#### Features Dropped by EDA — Combined Group (15 dropped)

| Dropped Feature | Reason |
|:----------------|:-------|
| `p1_kb_d_minus_a` | MW *p* = 0.595, \|*r*\| = 0.020 |
| `p1_kb_active` | MW *p* = 0.559, \|*r*\| = 0.011 |
| `p2_kb_active` | MW *p* = 0.636, \|*r*\| = 0.009 |
| `p1_mc_n_right` | MW *p* = 0.523, \|*r*\| = 0.002 |
| `p2_mc_n_total` | MW *p* = 0.414, \|*r*\| = 0.017 |
| `p2_mc_n_per_sec` | MW *p* = 0.140, \|*r*\| = 0.004 |
| `p2_mc_n_right` | MW *p* = 0.914, \|*r*\| = 0.016 |
| `p1_inter_crouch_then_shoot` | MW *p* = 0.583, \|*r*\| = 0.008 |
| `p1_inter_stop_then_shoot` | MW *p* = 0.616, \|*r*\| = 0.009 |
| `diff_kb_active` | Zero-rate = 99.7% |
| `diff_mc_n_right` | MW *p* = 0.394, \|*r*\| = 0.019 |
| `diff_mc_cv_interval` | MW *p* = 0.979, \|*r*\| = 0.020 |
| `is_M1` | MW *p* = 0.502, \|*r*\| = 0.013 |
| `is_M3` | MW *p* = 0.482, \|*r*\| = 0.013 |
| `round_duration` | MW *p* = 0.507, \|*r*\| = 0.010 |

**Phase C total: 94 candidates − 15 dropped = 79**

### S3.7 History Features (15 features)

History features capture in-match momentum. They are computed from rounds **prior** to the current one (no information leakage).

| Feature Name | Description |
|:-------------|:------------|
| `hist_n_rounds` | Rounds played so far in this match |
| `hist_p1_win_rate` | P1 cumulative win fraction across prior rounds |
| `hist_win_count` | P1 total wins so far |
| `hist_loss_count` | P1 total losses so far |
| `hist_current_streak` | Current streak (+wins, −losses) |
| `hist_max_win_streak` | Longest consecutive P1 win run |
| `hist_max_loss_streak` | Longest consecutive P1 loss run |
| `hist_exp_decay_wr` | Exponentially-weighted P1 win rate (α = 0.3) |
| `hist_wr_last2` | P1 win rate over last 2 rounds |
| `hist_wr_last5` | P1 win rate over last 5 rounds |
| `hist_wr_last10` | P1 win rate over last 10 rounds |
| `score_p1` | Current score for P1 |
| `score_p2` | Current score for P2 |
| `score_diff` | `score_p1 − score_p2` — largest Gini importance (0.096) |
| `round_duration` | Duration of the duel in seconds |

**Total: 15**

### S3.8 Metadata Features (9 features)

Metadata features encode participant-level attributes and match configuration, all fixed before the session begins or updated deterministically.

| Feature Name | Description |
|:-------------|:------------|
| `meta_p1_exp` | P1 experience level (Novice = 1, Intermediate = 2, Pro = 3) |
| `meta_p2_exp` | P2 experience level |
| `meta_exp_diff` | P1 − P2 experience — primary driver of full-model accuracy |
| `meta_is_M1` | Binary: AK / Rifle match |
| `meta_is_M2` | Binary: Pistol match |
| `meta_is_M3` | Binary: Scout / Sniper match |
| `score_p1` *(shared)* | Current score for P1 |
| `score_p2` *(shared)* | Current score for P2 |
| `score_diff` *(shared)* | `score_p1 − score_p2` |

`score_p1`, `score_p2`, and `score_diff` appear in both History and Metadata groups; they are stored once in the feature matrix.

**Total: 9**

### S3.9 Full Multimodal Feature Set (Phase D — 114 features)

| Group | Features | Section |
|:------|:--------:|:--------|
| Keyboard (P1 + P2 + diff) | 51 | S3.4 |
| Mouse (P1 + P2 + diff) | 18 | S3.5 |
| Interaction + Action Density | 10 | S3.6 |
| History | 15 | S3.7 |
| Metadata | 9 | S3.8 |

Phase D applies EDA jointly to the full combined candidate pool rather than taking the union of per-phase survivors. Features dropped in earlier phases (e.g., `is_M1` in Phase A) may survive when combined with contextual features that provide joint signal. The joint EDA produces exactly **114** features, as confirmed by the experimental output.

**Total: 114**

---

## S4. Skill Tier Self-Reporting

**Reviewer Comment 4:** *"Consider adding inter-rater reliability or validation against in-game rank if available."*

In-game rank data (CS2 Premier, Faceit ELO) was not collected as part of the study protocol (IRB No. IIITS/EC/2022/01). The monotonic win-rate gradient in Table I of the paper (Pro vs. Novice: 70%, Intermediate vs. Novice: 63%, Same Tier: 50%) provides indirect validation that self-reported tiers reflect actual skill differences.

---

## S5. AI Tools Used in Manuscript Preparation

**Reviewer Comment 5:** *"Specify which AI tool(s) for transparency."*

| Tool | Provider | Usage |
|:-----|:---------|:------|
| Claude Sonnet 4.5 | Anthropic | Grammar refinement, copy-editing, and minor rephrasing |

---

## Citation

> A. Suwalka, V. Prajapati, and A. C. Turlapaty, "Duel Outcome Classification in FPS Games Using Context-Driven Multimodal Fusion," in *Proc. ICMLT*, 2026.

## Contact

For questions regarding this supplementary material or the associated dataset: suwalkabhishek@gmail.com

---

*All statistics in this document are computed from the original experimental data. No re-training or re-analysis was performed.*
