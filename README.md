# Surrogate Modeling of a Binary Distillation Column Using DWSIM and Machine Learning

## Screening Task 3 – FOSSEE DWSIM

This project develops a machine-learning surrogate model for a rigorous binary
distillation column simulated in DWSIM.

The objective is to replace repeated rigorous DWSIM simulations with a fast
data-driven model capable of predicting important distillation performance
variables from operating conditions.

The selected system is a Benzene–Toluene binary mixture, simulated using the
Peng–Robinson (PR) thermodynamic property package.

---

## 1. Project Objective

The objective of this project is to develop and evaluate surrogate models that
can predict the following distillation-column outputs:

- Distillate benzene purity (xD)
- Bottoms benzene purity (xB)
- Condenser duty (QC)
- Reboiler duty (QR)

from selected operating conditions.

The machine-learning surrogate provides significantly faster predictions than
running a rigorous DWSIM simulation for every new operating point.

---

## 2. System Description

### Binary System

- Component 1: Benzene
- Component 2: Toluene

### Thermodynamic Model

- Peng–Robinson (PR)

### Simulation Software

- DWSIM

The DWSIM flowsheet contains a rigorous Distillation Column unit operation
with feed, distillate and bottoms streams.

Operating conditions were varied and the flowsheet was solved for each
operating point. Only successfully solved and physically valid simulation
cases were retained for machine-learning development.

---

## 3. Machine Learning Problem Formulation

### Input Variables

The surrogate model uses the following seven input variables:

| Variable | Description | Unit |
|---|---|---|
| Feed_T_K | Feed temperature | K |
| Feed_P_Pa | Feed pressure | Pa |
| Feed_xBenzene | Benzene mole fraction in feed | mol/mol |
| Number_of_Stages | Total number of column stages | - |
| Feed_Stage | Feed stage location | - |
| Reflux_Ratio | Reflux ratio | - |
| Bottoms_Flow_mol_s | Bottoms withdrawal rate | mol/s |

### Output Variables

| Variable | Description | Unit |
|---|---|---|
| xD_Benzene | Benzene mole fraction in distillate | mol/mol |
| xB_Benzene | Benzene mole fraction in bottoms | mol/mol |
| QC_kW | Condenser duty magnitude | kW |
| QR_kW | Reboiler duty magnitude | kW |

Feed molar flow was approximately fixed at:

10.00057 mol/s

because feed molar flow was not selected as an ML input variable.

---

## 4. Dataset Generation

The dataset was generated directly from rigorous DWSIM simulations.

Different operating conditions were applied to the existing Benzene–Toluene
distillation column, including:

- Feed temperature
- Feed pressure
- Feed composition
- Number of stages
- Feed stage location
- Reflux ratio
- Bottoms withdrawal rate

For each operating point:

1. The DWSIM column conditions were modified.
2. The feed conditions were updated.
3. The rigorous distillation calculation was executed.
4. The convergence status was checked.
5. Successful cases were recorded.
6. Distillate and bottoms compositions were extracted.
7. Condenser and reboiler duties were extracted.
8. Failed or physically invalid cases were excluded.

The final dataset contains unique successful DWSIM simulations after
preprocessing and duplicate removal.

---

## 5. Dataset

The main dataset is:

`Dataset/Dataset.csv`

The final modeling dataset contains:

- 7 input variables
- 4 target variables
- DWSIM-generated simulation results
- Successful operating points only
- No missing values after preprocessing
- No duplicate operating points after preprocessing

The dataset also retains the source-file information where applicable so that
the origin of simulation cases can be traced.

---

## 6. Operating Domain

The surrogate model is intended to be used only within the operating domain
represented by the successful DWSIM simulations.

The final dataset covers approximately:

| Variable | Minimum | Maximum |
|---|---:|---:|
| Feed Temperature | 350.1 K | 384.9 K |
| Feed Pressure | 95,020 Pa | 110,000 Pa |
| Feed Benzene Fraction | 0.303 | 0.700 |
| Number of Stages | 10 | 18 |
| Feed Stage | 4 | 16 |
| Reflux Ratio | 1.507 | 3.492 |
| Bottoms Flow | 4.005 mol/s | 5.999 mol/s |

Output ranges are approximately:

| Output | Minimum | Maximum |
|---|---:|---:|
| xD Benzene | 0.537 | 0.996 |
| xB Benzene | 0.0059 | 0.444 |
| QC | 0.334 kW | 0.811 kW |
| QR | 0.053 kW | 0.814 kW |

These ranges define the practical domain of the trained surrogate.

The model should not be interpreted as validated outside this domain.

---

## 7. Data Preprocessing

The following preprocessing steps were performed:

1. Loaded DWSIM-generated CSV files.
2. Combined compatible simulation datasets.
3. Removed duplicate operating points.
4. Removed unsuccessful simulation cases.
5. Checked for missing values.
6. Checked numerical data types.
7. Verified physical validity of purity values.
8. Verified non-negative duty magnitudes.
9. Separated input and target variables.
10. Split the data into training and testing subsets.

The train/test split used:

- Training: 80%
- Testing: 20%
- Random state: 42

Standardization was applied where required, particularly for:

- Linear Regression
- Artificial Neural Network

Tree-based models were trained without requiring feature standardization.

---

## 8. Machine Learning Models

Four machine-learning approaches were implemented and compared.

### 8.1 Linear Regression

Linear Regression was used as the baseline model.

It provides a simple reference for determining whether nonlinear machine-learning
models provide meaningful improvement.

---

### 8.2 Random Forest

Random Forest was used to model nonlinear relationships between operating
conditions and column outputs.

It is robust to nonlinear interactions and does not require feature
standardization.

---

### 8.3 XGBoost

XGBoost was used as the primary gradient-boosting model.

The final model used approximately:

- 300 estimators
- Maximum depth: 4
- Learning rate: 0.05
- Subsample: 0.8
- Column subsampling: 0.8

XGBoost was selected as the final surrogate based on its overall prediction
accuracy, robustness and physical consistency.

---

### 8.4 Artificial Neural Network

A feed-forward Artificial Neural Network was also evaluated.

The network used hidden layers containing:

- 64 neurons
- 32 neurons

The ANN used:

- ReLU activation
- Adam optimization
- Early stopping
- Standardized input features

---

## 9. Model Evaluation

Each model was evaluated independently for all four target variables.

The following metrics were used:

### Mean Absolute Error (MAE)

Measures the average absolute prediction error.

Lower MAE indicates better performance.

### Root Mean Squared Error (RMSE)

Measures the magnitude of prediction errors while giving greater weight to
larger errors.

Lower RMSE indicates better performance.

### R² Score

Measures the proportion of target variance explained by the model.

Higher R² indicates better predictive performance.

---

## 10. Test-Set Results

The final test-set comparison showed that XGBoost provided the strongest
overall performance.

| Model | Target | MAE | RMSE | R² |
|---|---|---:|---:|---:|
| Linear Regression | xD | 0.0433 | 0.0501 | 0.8684 |
| Linear Regression | xB | 0.0442 | 0.0508 | 0.8060 |
| Linear Regression | QC | 0.0162 | 0.0209 | 0.9856 |
| Linear Regression | QR | 0.0571 | 0.0673 | 0.9359 |
| Random Forest | xD | 0.0322 | 0.0477 | 0.8807 |
| Random Forest | xB | 0.0346 | 0.0484 | 0.8244 |
| Random Forest | QC | 0.0401 | 0.0506 | 0.9161 |
| Random Forest | QR | 0.0744 | 0.1134 | 0.8182 |
| XGBoost | xD | 0.0282 | 0.0387 | 0.9214 |
| XGBoost | xB | 0.0188 | 0.0245 | 0.9548 |
| XGBoost | QC | 0.0160 | 0.0200 | 0.9869 |
| XGBoost | QR | 0.0433 | 0.0772 | 0.9157 |
| ANN | xD | 0.0456 | 0.0596 | 0.8138 |
| ANN | xB | 0.0591 | 0.0905 | 0.3845 |
| ANN | QC | 0.0458 | 0.0587 | 0.8870 |
| ANN | QR | 0.0562 | 0.0688 | 0.9330 |

XGBoost achieved the strongest overall test-set performance, with an average
R² of approximately 0.939 across the four targets.

---

## 11. Cross-Validation

Five-fold cross-validation was performed to evaluate robustness and
generalization.

The XGBoost model achieved approximately:

| Target | Mean R² | Std R² | Mean MAE | Mean RMSE |
|---|---:|---:|---:|---:|
| xD_Benzene | 0.9091 | 0.0382 | 0.0296 | 0.0397 |
| xB_Benzene | 0.9098 | 0.0138 | 0.0264 | 0.0359 |
| QC_kW | 0.9739 | 0.0150 | 0.0202 | 0.0271 |
| QR_kW | 0.9399 | 0.0440 | 0.0386 | 0.0572 |

The cross-validation results indicate that the XGBoost surrogate generalizes
well within the represented DWSIM operating domain.

---

## 12. Feature Importance

Feature importance analysis was performed for the final XGBoost model.

The most influential feature was:

1. Reflux Ratio

followed by important operating variables such as:

2. Feed Temperature
3. Feed Benzene Composition
4. Bottoms Withdrawal Rate

The dominance of reflux ratio is physically reasonable because reflux directly
affects internal liquid-vapor traffic, separation efficiency and energy
requirements in a distillation column.

---

## 13. Physical Consistency

Physical consistency was explicitly checked in addition to statistical
accuracy.

The DWSIM dataset contains physically meaningful purity values satisfying:

0 <= xD <= 1

and

0 <= xB <= 1

The final surrogate was also evaluated for predictions outside the physical
purity range.

XGBoost showed substantially better physical stability than the less suitable
models.

For production use, predicted purity values should still be constrained or
post-processed to remain within the physical [0,1] interval.

---

## 14. Visual Analysis

The `Plots/` directory contains the generated analysis figures.

Important plots include:

- Correlation matrix
- Target distributions
- Condenser duty vs number of stages
- Reboiler duty vs number of stages
- Bottoms benzene fraction vs stages
- Distillate benzene fraction vs stages
- MAE comparison
- R² comparison
- RMSE comparison
- Model prediction plots
- Feature importance plots
- Prediction/trend analysis

These plots are used to analyze model accuracy, relationships between
variables and physical trends.

---

## 15. Final Model Selection

### Selected Model: XGBoost

XGBoost was selected as the final surrogate model because it provides the
best overall combination of:

- Prediction accuracy
- Nonlinear modeling capability
- Cross-validation robustness
- Generalization
- Physical consistency
- Feature interpretability

Its test-set R² values were approximately:

- xD_Benzene: 0.9214
- xB_Benzene: 0.9548
- QC_kW: 0.9869
- QR_kW: 0.9157

The average test-set R² was approximately:

0.9388

Therefore, XGBoost is used as the final surrogate model.

---

## 16. Repository Structure

The project is organized as follows:

```text
FOSSEE_DWSIM_ScreeningTask3/
│
├── Code/
│   ├── *.ipynb
│   ├── DWSIM scripts
│   └── helper scripts
│
├── DWSIM/
│   ├── Benzene_Toluene_Distillation_R2.dwxmz
│   └── other DWSIM flowsheet files
│
├── Dataset/
│   └── Dataset.csv
│
├── Plots/
│   ├── 01_correlation_matrix.png
│   ├── 02_target_distributions.png
│   ├── 03_QC_kW_vs_stages.png
│   ├── 03_QR_kW_vs_stages.png
│   ├── 03_xB_Benzene_vs_stages.png
│   ├── 03_xD_Benzene_vs_stages.png
│   ├── 05_MAE_comparison.png
│   ├── 05_R2_comparison.png
│   └── 05_RMSE_comparison.png
│
├── Results/
│   ├── Cross_Validation_Results.csv
│   ├── Model_Performance_Comparison.csv
│   ├── Model_Predictions.csv
│   └── Model_Ranking.csv
│
├── Trained Models/
│   ├── linear_regression.joblib
│   ├── random_forest.joblib
│   ├── xgboost.joblib
│   └── ann.joblib
│
├── Report.pdf
├── Results_Summary.pdf
└── README.md
17. How to Reproduce the Work
Step 1 – Install DWSIM

Install DWSIM on a Windows system.

Open the supplied .dwxmz flowsheet using DWSIM.

Step 2 – Open the DWSIM Flowsheet

Open:

DWSIM/Benzene_Toluene_Distillation_R2.dwxmz

Verify that the Benzene–Toluene mixture and the rigorous distillation column
are present.

The flowsheet should contain:

Feed stream
Distillate stream
Bottoms stream
Distillation column
Required thermodynamic property package
Step 3 – Generate Simulation Data

The DWSIM Python scripts in the Code/ directory automate the simulation
process.

The scripts modify selected operating conditions, solve the flowsheet and
record successful simulation results.

The generated CSV files should be placed in the dataset directory.

Failed DWSIM simulations should not be included in the final training data.

Step 4 – Run the Jupyter Notebook

Open the notebook located inside:

Code/

Run the notebook sequentially from the first cell to the last cell.

The notebook performs:

Dataset loading
Dataset cleaning
Duplicate removal
Exploratory data analysis
Correlation analysis
Train/test splitting
Feature preprocessing
Model training
Model evaluation
Cross-validation
Feature importance analysis
Physical consistency checks
Prediction generation
Plot generation
Final model comparison
Step 5 – Train the Models

The notebook trains:

Linear Regression
Random Forest
XGBoost
Artificial Neural Network

The trained models are saved into:

Trained Models/
Step 6 – Evaluate the Models

The notebook generates:

Results/

containing model comparison, prediction and cross-validation results.

It also generates the plots inside:

Plots/
Step 7 – Reproduce the Final XGBoost Model

The final selected model is:

Trained Models/xgboost.joblib

It can be loaded using joblib and used for rapid prediction without running
the full DWSIM simulation.

Example:

import joblib

model = joblib.load("Trained Models/xgboost.joblib")

prediction = model.predict(new_operating_conditions)

print(prediction)

The input operating conditions must remain within the domain represented by
the training dataset.

18. Important Assumptions

The following assumptions were made:

The system is a binary Benzene–Toluene mixture.
The Peng–Robinson thermodynamic model is used.
Feed molar flow is approximately fixed at 10.00057 mol/s.
Only successfully converged DWSIM simulations are used.
Duplicate operating points are removed.
Condenser and reboiler duties are represented as positive magnitudes.
The surrogate is intended for interpolation within the simulation domain.
Predictions outside the trained operating domain should not be considered
validated.
Purity outputs are physically constrained to the range [0,1] for production
use.
19. Limitations

The surrogate model is data-driven and therefore depends on the quality and
coverage of the DWSIM simulation dataset.

It should not be considered a replacement for rigorous DWSIM simulation outside
the operating region represented in the training data.

The feed molar flow was fixed and therefore the current surrogate does not
predict the effect of changes in feed flow.

Additional stage configurations were attempted during simulation generation,
but only successfully converged cases were retained in the final modeling
dataset.

Consequently, the model should not be extrapolated to stage counts or
operating conditions that are not represented in the final dataset.

20. Key Results

The final analysis demonstrates that machine learning can effectively act as a
surrogate for the rigorous Benzene–Toluene DWSIM distillation model.

The main findings are:

XGBoost provided the strongest overall predictive performance.
Condenser duty was predicted with particularly high accuracy.
Reflux ratio was the most influential feature.
Feed temperature and feed composition also had significant influence.
Five-fold cross-validation supported the robustness of XGBoost.
ANN showed comparatively weaker generalization, particularly for bottoms
purity.
Physical consistency checks were incorporated into the evaluation.
The final surrogate is suitable for rapid prediction within the represented
DWSIM operating domain.
21. Final Conclusion

A rigorous Benzene–Toluene binary distillation model was successfully used in
DWSIM to generate simulation data for machine-learning surrogate modeling.

Four machine-learning methods were evaluated:

Linear Regression
Random Forest
XGBoost
Artificial Neural Network

Among these models, XGBoost achieved the best overall balance of accuracy,
robustness and physical consistency.

Therefore, XGBoost is selected as the final surrogate model.

The developed surrogate can provide rapid estimates of:

Distillate benzene purity
Bottoms benzene purity
Condenser duty
Reboiler duty

without requiring a full rigorous DWSIM calculation for every prediction.

However, the surrogate should be used only within the operating domain covered
by the successful DWSIM simulations.

22. Reproducibility Checklist

Before submission, verify that the compressed project folder contains:

 Report.pdf
 Results_Summary.pdf
 Dataset.csv
 DWSIM .dwxmz flowsheet
 Jupyter notebook
 DWSIM automation/helper scripts
 Generated result CSV files
 Generated plots
 Trained model files
 README.md

The complete project is intended to be reproducible by opening the DWSIM
flowsheet, generating/using the simulation dataset, and executing the provided
notebook.

Author

Lavanya Saxena

B.Tech – Artificial Intelligence & Data Science

Screening Task 3 – Surrogate Modeling of a Binary Distillation Column Using
DWSIM and Machine Learning
