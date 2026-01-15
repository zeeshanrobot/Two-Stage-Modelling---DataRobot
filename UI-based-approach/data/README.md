# Data Description

This project uses an insurance claims dataset.

Raw data is not included in this repository due to confidentiality constraints.

## Expected Columns

### Stage 1 – Fraud Detection
- `fraud_reported` (target)
- Policy, customer, and incident-related features

### Stage 2 – Claim Severity
- `total_claim_amount` (target)
- Same feature set as Stage 1
- External prediction column from Stage 1:
  - `fraud_reported_1_PREDICTION`

## Notes
- Stage 2 training data contains only non-fraudulent claims
- External prediction columns must be present at inference time
