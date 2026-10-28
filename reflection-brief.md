# Capstone Reflection Brief

## 1. Insurance Policy Extraction Pipeline

**Evidence:** `insurance/01-policy-pipeline-tests.txt`, `insurance/04-calibration-report.txt`, `insurance/05-offline-routing-tests.txt`, `insurance/06-retry-perturbation.txt`, `insurance/07-premium-validation-perturbation.txt`, `insurance/08-routing-decision-demo.txt`, and `insurance/08-routing-decision-demo.json`.

The main test suite recorded 45 passing tests and 3 skipped tests. The offline routing suite recorded 9 passing tests. The retry suite recorded 13 passing tests and 1 skipped live test. Mypy and Ruff both passed.

The routing tests demonstrate that low confidence, integration failures, and reviewer disagreement can lead to human review. A deterministic routing demonstration using the project's actual `route_extraction` function produced a `human_review` decision for synthetic record `DEMO-POL-LOW-CONFIDENCE`. Its `premium_amount` confidence was 0.65, below the configured threshold of 0.90. The saved JSON records the decision and its reason.

The premium-validation perturbation used a manually constructed baseline with a $2,350.00 premium and components totaling $2,350.00. Changing only the stated premium to $2,500.00 triggered `premium_does_not_match_components` and a reported discrepancy of $150.00.

The calibration experiment shows why subgroup metrics matter: the umbrella/exclusions slice had confidence 0.93, accuracy 0.00, and Brier score 0.865, while the overall Brier score was 0.291. The aggregate score alone would not clearly communicate this subgroup's weakness.

**Limitations:** The routing demonstration is synthetic, not a decision produced by extracting one of the bundled policies. The premium-validation experiment uses manually constructed validator inputs rather than a modified policy document. A full end-to-end run over the bundled policy documents has not been captured because API access is not configured. Therefore, the evidence does not establish that the complete Insurance pipeline generated routing decisions for the bundled policies.

## 2. Mortgage Document Extraction System

**Evidence:** `mortgage/04-missing-bonus-run.txt`, `mortgage/05-appraisal-normalization-run.txt`, and `mortgage/06-income-mismatch-run.txt`.

The missing-bonus run returned `bonus_monthly: null` rather than inventing a value. The appraisal run normalized the informal square-footage value to integer 2400 and returned `consistent: true`.

The income mismatch run calculated 9642.17 while the stated total was 10892.17. It reported a delta of -1250.00 and marked the extraction inconsistent. This demonstrates the value of validating arithmetic rather than trusting a structured extraction without checking it.

The comparison uses supplied fixtures rather than a deliberately edited input. A separate input-perturbation experiment would strengthen this section.

## 3. Supply Chain Risk Investigation

**Evidence:** `supply-chain/report-normal.md`, `supply-chain/report-timeout.md`, `supply-chain/report-normal.log`, and `supply-chain/report-timeout.log`.

The normal investigation generated a briefing. The simulated-timeout run completed despite a logistics-source failure and retained partial results, marking unavailable information in the Incomplete section.

The normal and timeout reports demonstrate that source failure should reduce coverage transparently rather than silently fabricate information or abort the entire investigation. The experiment changes the execution mode rather than modifying source data.

The briefing also preserves conflicting on-time-delivery claims, approximately 95% and 78%, instead of collapsing them into a single unsupported value. Keeping both values with provenance allows a reviewer to investigate the discrepancy.

## 4. Reliability Principles Across the Three Systems

All three systems demonstrate the importance of making uncertainty and failure visible.

- The Insurance pipeline uses validation, retry rules, routing, and calibration slices to identify outputs needing additional scrutiny.
- The Mortgage system checks structured extraction against arithmetic constraints and preserves missing values as null.
- The Supply Chain system preserves source attribution and partial results when sources conflict or become unavailable.

The reliability principle I would apply in my own work is: **validate outputs, preserve provenance, and escalate uncertainty instead of hiding it behind a confident-looking result.** Automated systems should make it easy for a reviewer to understand what was observed, what failed, and what still needs investigation.

## 5. Verification Summary

- Insurance: 45 passed, 3 skipped in the full suite; 9 routing tests passed; 13 retry tests passed and 1 skipped.
- Mortgage: 25 tests passed.
- Supply Chain: 34 tests passed.
- Mypy and Ruff passed for all three systems.
- Environment: Python 3.13.0 on Linux, recorded in `environment.txt`.

## 6. Remaining Submission Checks

1. Run the full Insurance pipeline over the bundled policy documents and capture its genuine routing output if API access becomes available. The synthetic routing demonstration is not a substitute for this run.
2. Verify the Mortgage README's reference to 28 acceptance tests against the 25 tests discovered.
3. Capture at least one screenshot per system if required by the submission instructions.
4. Confirm the final submission folder matches the starter README layout.

### Insurance Confidence-Threshold Perturbation

Evidence: `insurance/09-threshold-perturbation.txt`.

Using the project's actual `route_extraction()` function, the same synthetic extraction record was tested with confidence thresholds of 0.90 and 0.96. At 0.90, the decision was `auto_approve`; at 0.96, it changed to `human_review` because all six field confidence scores were below the threshold.

This demonstrates deterministic routing behavior when the configured confidence threshold changes. **Limitation:** The input was synthetic. This experiment does not establish that the full pipeline ran on the bundled policy documents or generated routing decisions from that run.
