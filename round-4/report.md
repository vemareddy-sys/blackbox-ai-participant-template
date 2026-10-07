GK-02 Black-Box Reconstruction — Round 4

1. Objective

The objective was to reconstruct the behavior of the GK-02 hospital triage black-box system using observations collected during the previous reverse-engineering rounds.

The system accepts 10 input variables:

- "age"
- "baseline_score"
- "comorbidity_ratio"
- "dependants"
- "prior_visits"
- "recent_admissions"
- "requested_beds"
- "vitals_index"
- "ward"
- "years_registered"

It returns:

- "score": a value between 0 and 1
- "decision": "APPROVE" or "DECLINE"

The reconstruction was implemented as a data-driven surrogate rather than claiming recovery of the original hidden implementation.

2. Observed Data

The notebook contains 117 verified GK-02 observations collected from the black-box system.

The observations were embedded directly into the reconstruction notebook and converted into a tabular dataset for analysis.

The observed outputs show that the decision is not determined solely by one input. Multiple numerical variables and categorical ward information contribute to the resulting score.

3. Reconstruction Strategy

The reconstruction followed four stages:

1. Collect and preserve the verified black-box input/output observations.
2. Encode the categorical "ward" variable numerically.
3. Construct interaction features to capture non-linear relationships between important variables.
4. Train and compare multiple tabular regression/classification models and combine their predictions into a surrogate ensemble.

The notebook also evaluates feature importance to identify variables that explain the observed score variation.

The reconstruction uses:

- scikit-learn models
- XGBoost
- LightGBM
- CatBoost
- interaction features
- weighted ensemble modelling
- stacking

The notebook was designed to run in Google Colab and supports both CPU execution and GPU execution.

4. Important Behavioral Findings

4.1 Score is continuous

The black-box does not simply return a binary result. It produces a continuous score in the range 0–1 and then provides a decision.

Examples in the collected observations include:

- "0.9207"
- "0.9372"
- "0.9769"
- "0.8079"
- "0.5777"

This indicates that the score contains information beyond the final binary decision.

4.2 Decision boundary was not observed in the supplied observations

All embedded observations shown in the reconstruction dataset have the decision "APPROVE".

Therefore, the supplied evidence is insufficient to determine the exact threshold separating "APPROVE" and "DECLINE".

The reconstruction therefore treats the decision boundary as an unresolved component rather than inventing a threshold.

4.3 Recent admissions has a measurable effect

Several controlled observations change "recent_admissions" while keeping most other inputs fixed.

For example, one sequence changes the value from approximately "0.8" to "2.56", with the score changing from approximately "0.9768" to "0.9541".

This supports a negative relationship between recent admissions and the score in that region.

4.4 Prior visits has a measurable effect

A controlled change in "prior_visits" also produces a substantial score change.

For example, the observations include:

- "prior_visits = 20.0" → score approximately "0.9768"
- "prior_visits = 5.8" → score approximately "0.9357"
- "prior_visits = 2.0" → score approximately "0.7519"

This suggests that prior visits can have significant influence on the score.

4.5 Age influences the score

The collected observations include controlled changes in age while other variables remain largely fixed.

For example, scores decrease as age is moved from approximately 75 toward younger values in one observed sequence:

- age "75.0" → approximately "0.9761"
- age "68.73" → approximately "0.9755"
- age "52.485" → approximately "0.9622"
- age "18.0" → approximately "0.9063"

Therefore age is not ignored by the black box.

4.6 Some variables appear to have conditional or weak effects

Several observations show little or no visible score change after changing individual variables such as "requested_beds" or "vitals_index" within particular regions of the input space.

This suggests that the hidden function may contain interactions, saturation, thresholds, or feature-dependent weighting rather than behaving as a simple linear formula.

5. Feature Importance

The reconstruction's feature-importance analysis identified the following features among the strongest contributors:

Feature| Importance
dependants| 0.210889
years_registered| 0.181366
comorbidity_ratio| 0.140128
baseline_x_comorbidity| 0.078500
age_x_baseline| 0.076930
ward_num| 0.061858
prior_visits| 0.055033
age| 0.042889

The strongest measured feature in this reconstruction was "dependants", followed by "years_registered" and "comorbidity_ratio".

Interaction terms were also important, particularly:

- "baseline_score × comorbidity_ratio"
- "age × baseline_score"

This supports the conclusion that modelling interactions is useful for approximating the black-box behavior.

6. Reconstruction Model

The reconstruction does not claim to reproduce the proprietary/internal source code of GK-02.

Instead, it approximates the observed mapping:

"10 input variables → score → decision"

using supervised learning and ensemble methods.

The model is trained only from the observed black-box input/output pairs. Consequently, predictions outside the observed input distribution should be treated as extrapolations.

7. Testing

Testing was performed by comparing reconstructed predictions against the collected observations.

The test process focuses on:

- score prediction error
- agreement with observed decisions
- behavior under changed individual variables
- interaction effects
- consistency across different model families

The reconstruction is considered successful when it reproduces the observed score behavior more closely than a simple baseline model.

8. Limitations

The main limitation is the available observation set.

Although 117 verified observations provide substantial behavioral evidence, they do not uniquely determine the hidden implementation.

In particular:

- the exact mathematical formula is not known;
- the exact "APPROVE"/"DECLINE" threshold is not established by the supplied observations;
- the observations do not uniformly cover the entire input space;
- correlations between variables can make individual feature effects difficult to isolate;
- machine-learning feature importance describes the surrogate model and should not automatically be interpreted as the exact importance used internally by GK-02.

9. Final Reconstruction Claim

The final reconstruction is a behavioral surrogate of GK-02 based on the observed black-box evidence.

The evidence supports a multi-variable, non-linear scoring process with meaningful contributions from demographic/history variables, comorbidity, dependants, registration duration, ward, and interaction effects.

The reconstruction intentionally avoids claiming exact recovery of the hidden source function where the collected observations do not provide sufficient evidence. 