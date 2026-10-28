# Perturbation Log

## 1. Insurance Policy Extraction Pipeline

### Experiment A: Retry and Failure Behavior

**Command:** `.venv/bin/pytest tests/test_us01_retry.py -v`

**Evidence:** `insurance/06-retry-perturbation.txt`

**Observed result:** 13 passed, 1 skipped. Tests cover format and consistency failures that trigger retries, missing-source behavior that halts immediately, and retry metadata. The live extraction test was skipped.

**Comparison:** A well-formed response is accepted; format or consistency failures can trigger a retry, while missing required source information halts immediately. This experiment uses the provided tests rather than modifying a policy document.

### Experiment B: Controlled Premium-Validation Perturbation

**Evidence:** `insurance/07-premium-validation-perturbation.txt`

**Method:** Constructed a baseline validator input with a stated premium of $2,350.00 and components totaling $2,350.00. Then changed only the stated premium to $2,500.00.

**Observed result:** The baseline returned `None` (no validation error). The perturbed input returned a consistency error with pattern `premium_does_not_match_components` and a reported discrepancy of $150.00.

**Comparison:** The validator accepted the internally consistent baseline and detected the mismatch after the stated premium changed.

**Limitation:** This was a manually constructed validator-input experiment. It did not modify the original policy document or demonstrate a full end-to-end extraction and routing run. The baseline is a normalized test case, not the original document's stated premium.

### Experiment D: Confidence-Threshold Perturbation

**Evidence:** `insurance/09-threshold-perturbation.txt`

**Method:** Used the same synthetic extraction record, with every field confidence fixed at 0.95, and changed the routing threshold from 0.90 to 0.96.

**Command:** A Python script called the project's actual `route_extraction()` function with each threshold.

**Observed result:** At 0.90, the record was `auto_approve`. At 0.96, the record was `human_review`, because all six field confidences were below the new threshold.

**Comparison:** Raising the threshold changed the routing decision for the same input, demonstrating that routing behavior responds deterministically to the configured confidence threshold.

**Limitation:** This is a synthetic routing-configuration experiment, not a run on a modified bundled policy document or a full end-to-end pipeline run.

### Experiment C: Deterministic Human-Review Routing Demonstration

**Evidence:**
- `insurance/08-routing-decision-demo.txt`
- `insurance/08-routing-decision-demo.json`

**Method:** Created a synthetic policy-extraction record and passed it through the project's actual `route_extraction` function, with agreeing reviewer results and passing integration findings.

**Observed result:** The routing function returned `human_review` because `premium_amount` had confidence 0.65, below the configured threshold of 0.90. The JSON output records the decision, reason, confidence summary, and field below the threshold.

**Comparison:** The demonstration confirms that the routing function sends a low-confidence record to human review even when the reviewer agrees and the integration findings pass.

**Limitation:** This is a synthetic routing demonstration, not a decision produced by extracting a bundled policy document. It does not establish that the full end-to-end pipeline ran.

## 2. Mortgage Document Extraction System

**Experiment:** Compare clean square-footage normalization with an income-total mismatch.

**Commands:**
- `.venv/bin/mortgage-extract fixtures/documents/appraisal_informal_sqft.txt --mode replay`
- `.venv/bin/mortgage-extract fixtures/documents/income_sum_mismatch.txt --mode replay`

**Evidence:**
- `mortgage/05-appraisal-normalization-run.txt`
- `mortgage/06-income-mismatch-run.txt`

**Observed result:** The appraisal's gross living area is normalized to integer 2400 and validation is consistent. For the income document, the calculated total is 9642.17 while the stated total is 10892.17; the validator reports a delta of -1250.00 and marks the result inconsistent.

**Comparison:** The appraisal is accepted without discrepancies, while the income total triggers a discrepancy. This is a comparison of two provided fixtures, not a modification of either input.

## 3. Supply Chain Risk Investigation

**Experiment:** Simulate a logistics-source timeout.

**Commands:**
- `.venv/bin/supply-chain-investigate meridian --offline`
- `.venv/bin/supply-chain-investigate meridian --offline --simulate-timeout`

**Evidence:**
- `supply-chain/report-normal.md`
- `supply-chain/report-timeout.md`
- `supply-chain/report-normal.log`
- `supply-chain/report-timeout.log`

**Observed result:** The normal run generates the briefing. The timeout run completes with the logistics source marked unavailable and missing information reflected in the Incomplete section.

**Comparison:** The normal run can use logistics data; the simulated-timeout run preserves partial results and reports the source failure rather than aborting. This changes the execution mode, not the source files.