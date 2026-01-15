# Two-Stage Insurance Claim Modeling using DataRobot

## Overview

This repository demonstrates an end-to-end **two-stage machine learning workflow** for insurance claims using **DataRobot**.

The solution mirrors real-world insurance operations by separating:
1. **Fraud Detection** (Classification)
2. **Claim Severity Estimation** (Regression)

Each stage is trained, deployed, and evaluated independently, then connected using **deployment-based inference** to simulate a realistic production workflow.

Link to presentation - [https://docs.google.com/presentation/d/191qN8cPv6ZuI45kZGJM1hZqdaG9P0icbomqhK364jZ8/edit?slide=id.g3b29a8c2362_0_4050#slide=id.g3b29a8c2362_0_4050](url)

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
4. Select the target column:                                                                                                                                                 <img width="953" height="413" alt="1_target_selection" src="https://github.com/user-attachments/assets/e9b0613a-9d70-4e83-8319-929111438b31" />
 
5. Confirm the project type as **Classification**
6. Enable:                                                                                                                                                                  Go to additional setting and select CV with N folds and select start modelling. Alter the metrics of validation of your choice                                             <img width="953" height="398" alt="2_image" src="https://github.com/user-attachments/assets/46f9365e-513b-4356-93c8-3a3fcbfb5c3e" />

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
 <img width="953" height="409" alt="3_img" src="https://github.com/user-attachments/assets/bce7eb2e-c3f3-4a09-a340-d76054f17c75" />


> Play with different threshold values to identify the balance between Precision and Recall
 <img width="920" height="404" alt="4_img" src="https://github.com/user-attachments/assets/66db53f2-4967-46a1-aafa-217ccbdf4752" />

> Once ok with all metrics deploy that model inside datarobot and choose the threshold value for the model
 <img width="956" height="353" alt="5_img" src="https://github.com/user-attachments/assets/d77f77eb-ca08-4197-8756-0e3b6dd78fca" />


---

### Stage 2 Project – Claim Severity Estimation (Regression)

**Objective:** Estimate claim amounts for legitimate claims only.

## External Predictions for Stage 2 (Regression)

For the claim severity model (Stage 2), predictions are generated using **DataRobot External Predictions** via a deployed model.

Unlike training-time evaluation, external predictions simulate **real production inference**, where the model receives new, unseen data and returns predictions without access to target values.

### Why External Predictions Are Used

External predictions are used in Stage 2 to:

- Ensure **production-aligned inference**
- Decouple model scoring from training projects
- Apply the regression model only to claims predicted as **non-fraudulent** in Stage 1
- Validate end-to-end behavior using deployed models

This approach mirrors how claim severity models are consumed in real insurance systems.

### How External Predictions Are Performed

1. Claims predicted as **non-fraudulent** by the Stage 1 deployment are selected
2. Target columns are removed from the input data
3. The filtered dataset is submitted to the **Stage 2 deployment** using the Batch Prediction API
4. The deployment returns predicted claim amounts in the `<target>_PREDICTION` column
5. Predictions are joined back to the original records for analysis and evaluation

### Output from Stage 2 Deployment

The regression deployment returns predictions in the following format:

- `total_claim_amount_PREDICTION` – Predicted claim amount
- Deployment metadata (approval status, timestamps, etc.)

No manual thresholding is applied in Stage 2, as regression models return continuous numeric outputs.

### Key Benefits of This Approach

- Uses the **same scoring mechanism as production**
- Ensures consistency between offline evaluation and live inference
- Enables independent governance and monitoring of each stage
- Supports scalable, real-world deployment patterns

By using external predictions for Stage 2, the solution demonstrates a fully governed and production-ready two-stage modeling workflow in DataRobot.


#### Data Preparation (Before Project Creation)

Filter the training dataset to include only non frauddelent cases


This ensures the regression model is trained exclusively on non-fraudulent claims.

#### Steps in DataRobot UI

1. Create a **new DataRobot project**
2. Upload the filtered dataset ( non fraudlent cases)
3. Select the target column:                                                                                                                                                 <img width="1903" height="880" alt="img_1 (1)" src="https://github.com/user-attachments/assets/aae95e29-37fa-484c-b08d-f69c63d38abf" />                                                                                                                                                                                                   
4. We can also create list of feature that we want to use to build model
5. Before starting Autopilot, open **Advanced Options / Additional Settings**
   <img width="1888" height="877" alt="2_advanced_option_feature_list" src="https://github.com/user-attachments/assets/f75d1acc-32e2-465c-ac98-c3fd13fb7cba" />
 
7. Enable **External Predictions**
  <img width="1903" height="733" alt="3_external_prediction" src="https://github.com/user-attachments/assets/eb3f7e15-62aa-4f36-8099-cf76341f65ae" />

8. Select the external prediction column coming from Stage 1:                                                                                                              - This column represents the fraud probability generated by the Stage 1 classification model
 - It will be supplied at inference time from the Stage 1 deployment
7. Confirm the project type as **Regression**
8. Start **Autopilot / Experimentation**
  <img width="1903" height="697" alt="5_start_autopilot" src="https://github.com/user-attachments/assets/b20995de-6e4e-4de7-a806-591443ab26ba" />

9. Evaluate models using:
- RMSE
- MAE
10. Select the **Recommended for Deployment** model
11. Deploy the model
12. Capture the generated:
- **Project ID**
- **Deployment ID**

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




