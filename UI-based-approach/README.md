# Two-Stage Insurance Claim Modeling using DataRobot

## Overview

This repository demonstrates an end-to-end **two-stage machine learning workflow** for insurance claims using **DataRobot**.

The solution mirrors real-world insurance operations by separating:
1. **Fraud Detection** (Classification)
2. **Claim Severity Estimation** (Regression)

Each stage is trained, deployed, and evaluated independently, then connected using **deployment-based inference** to simulate a realistic production workflow.

---

## Business Problem

Insurance organizations must make two critical decisions for every incoming claim:

- Detect potentially **fraudulent claims** as early as possible  
- Estimate the **claim amount** only for legitimate (non-fraudulent) claims  

A single model cannot effectively address both objectives because fraud detection and claim severity estimation are fundamentally different modeling problems. To address this, a **two-stage modeling approach** is adopted.

---

## Solution Architecture

### Stage 1 – Fraud Detection (Classification)

- **Target column:** `fraud_reported`
- **Model type:** Binary Classification
- **Outputs:**
  - Probability that a claim is fraudulent
  - Final fraud decision (0 / 1)
- **Threshold handling:**  
  The fraud decision threshold is configured and enforced **inside the DataRobot deployment**, ensuring consistent and governed decisioning.

### Stage 2 – Claim Severity Estimation (Regression)

- **Target column:** `total_claim_amount`
- **Model type:** Regression
- **Input population:**  
  Only claims predicted as **non-fraudulent** by Stage 1
- **Output:**  
  Estimated total claim amount

All predictions for both stages are executed using **DataRobot Deployments** via the **Batch Prediction API**, providing production-aligned inference.

---

## DataRobot Project Setup (UI Workflow)

This solution uses **two independent DataRobot projects**, one for each modeling stage.

---

### Stage 1 Project – Fraud Detection (Classification)

**Objective:** Predict whether an insurance claim is fraudulent.

#### Steps in DataRobot UI

1. Log in to the **DataRobot UI**
2. Click **Create New Project**
3. Upload the insurance claims dataset
4. Select the target column:
5. Confirm the project type as **Classification**
6. Enable:
- Cross-validation
- Holdout partition (recommended ~20%)
7. Start **Autopilot / Experimentation**
8. Review model performance using:
- Log Loss
- AUC
- Precision and Recall
9. Select the **Recommended for Deployment** model
10. Configure a **business-appropriate threshold** for fraud detection
11. Deploy the model
12. Capture the generated:
 - **Project ID**
 - **Deployment ID**

> Screenshots in this section illustrate experiment configuration, leaderboard comparison, and deployment setup.

---

### Stage 2 Project – Claim Severity Estimation (Regression)

**Objective:** Estimate claim amounts for legitimate claims only.

#### Data Preparation (Before Project Creation)

Filter the training dataset to include only non frauddelent cases


This ensures the regression model is trained exclusively on non-fraudulent claims.

#### Steps in DataRobot UI

1. Create a **new DataRobot project**
2. Upload the filtered dataset
3. Select the target column:
4. Confirm the project type as **Regression**
5. Start **Autopilot / Experimentation**
6. Evaluate models using:
- RMSE
- MAE
7. Select the **Recommended for Deployment** model
8. Deploy the model
9. Capture the generated:
- **Project ID**
- **Deployment ID**

> Screenshots in this section show regression experiment setup and model evaluation metrics.

---

## End-to-End Inference Flow

1. Incoming claims are scored using the **Stage 1 deployment**
2. The deployment returns:
- Fraud probability
- Final fraud classification based on the configured threshold
3. Claims predicted as **non-fraudulent** are passed to **Stage 2**
4. The **Stage 2 deployment** estimates the total claim amount
5. Results can be evaluated on holdout data or used directly in production systems

This design closely reflects how real-world insurance platforms operate.

---


