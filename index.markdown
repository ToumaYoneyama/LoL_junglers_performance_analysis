---
layout: default
title: Do Junglers Really Deserve the Blame?
---

# 🎮 Do Junglers Really Deserve the Blame?

This project investigates whether Junglers are statistically responsible for a team’s loss using real match data from League of Legends. Scroll down for each section of the analysis.

---

## 🧭 Table of Contents
- [1. Introduction & Research Question](#introduction)
- [2. Data Cleaning](#data-cleaning)
- [3. Feature Exploration](#feature-exploration)
- [4. Baseline Model](#baseline-model)
- [5. Final Model + Evaluation](#final-model)
- [6. Conclusion](#conclusion)

---

## 🧩 1. Introduction & Research Question <a name="introduction"></a>

### Stating the Question
Looking at the data, it seems like there are enough statistics for me to finally explore a long-standing question: 'Are Junglers really the ones to blame?' As someone who only knows League of Legends through my friends and some media outlets, it seems like players often hate on Junglers simply because they need someone to blame. In other words, the reason why Junglers are perceived as trolls could be psychological, and not actually backed by evidence. So, I will take this opportunity to examine whether there is any correlation between a Jungler’s performance—relative to their teammates—and the outcome of the game. Or in other words, figure out whether Junglers performing poorly have a significant impact on the outcome of the game, compared to other roles.

### Why You Should Care
I will explore this questions in hopes to contribute to broader discussions in esports analytics about how performance should be measured fairly. However, at the core, if you are like one of my friends and blames the Junglers without a valid statistical reasoning, man up for the poor guy and give him a chance by reading this report.

### Data Set Summary
The dataset includes professional League of Legends match statistics from the 2024 season:

- Shape: (117636, 161)
- Relevant Columns:
    - `earnedgoldshare`: The proportion of team gold earned
    - `damageshare`: The proportion of team damage dealt
    - `kills`, `assists`, `teamkills`: Used to calculate kill participation [(kills + assists) / teamkills]
    - `total cs`: Used to calculate share of team's creep score
    - `deaths`, `teamdeaths`: Used to calculate share of team's deaths
    - `team_dragons`, `team_barons`, `team_towers`, `team_inhibitors`: Team objectives secured
    - `opp_team_dragons`, `opp_team_barons`, `opp_team_towers`, `opp_team_inhibitors`: Objectives lost to the opposing team

---

## 🧹 2. Data Cleaning <a name="data-cleaning"></a>

### Cleaning the Data
I filtered the dataset to only include rows where the role was 'jng', then selected a subset of columns related to Jungler performance. I engineered three new features:

- `kill_participation` = (kills + assists) / teamkills  
- `cs_participation` = total cs / team cs  
- `deathshare` = deaths / teamdeaths

There were no missing values in the final subset, so no imputation was needed.

### Glancing at the Cleaned Data
Head of Relevant Columns (besides meta data):

|   result |   earnedgoldshare |   damageshare |   kill_participation |   cs_participation |   deathshare |   team_dragons |   team_barons |   team_towers |   team_inhibitors |   opp_team_dragons |   opp_team_barons |   opp_team_towers |   opp_team_inhibitors |
|---------:|------------------:|--------------:|---------------------:|-------------------:|-------------:|---------------:|--------------:|--------------:|------------------:|-------------------:|------------------:|------------------:|----------------------:|
|        0 |          0.154069 |     0.176101  |             1        |           0.146692 |     0.25     |              2 |             0 |             2 |                 0 |                  3 |                 2 |                 9 |                     1 |
|        1 |          0.157264 |     0.0692822 |             0.8125   |           0.153636 |     0        |              3 |             2 |             9 |                 1 |                  2 |                 0 |                 2 |                     0 |
|        0 |          0.186598 |     0.056958  |             0.666667 |           0.189331 |     0.294118 |              0 |             0 |             2 |                 0 |                  4 |                 1 |                 9 |                     1 |
|        1 |          0.177672 |     0.181341  |             0.882353 |           0.170557 |     0.333333 |              4 |             1 |             9 |                 1 |                  0 |                 0 |                 2 |                     0 |
|        1 |          0.21582  |     0.139214  |             0.619048 |           0.206107 |     0.333333 |              2 |             1 |            10 |                 2 |                  1 |                 0 |                 0 |                     0 |


### Univariate Analysis
Overall, features were either already well normalized or skewed.

- Already Normalized Data:

<iframe
 src="/LoL_junglers_performance_analysis/assets/cs_participation_histogram.html"
 width="800"
 height="600"
 frameborder="0"
></iframe>

**Interpretation**: Distribution is not skewed. Most Junglers contribute between 15% and 25% of their team’s total creep score, which makes sense since there are 5 players and lane players typically get more minions. This supports its use in raw form without transformation.

- Skewed Data:

<iframe
 src="/LoL_junglers_performance_analysis/assets/damageshare_histogram.html"
 width="800"
 height="600"
 frameborder="0"
></iframe>

**Interpretation**: Right-skewed distribution. A small number of Junglers contribute disproportionately high damage, likely due to champion pick (e.g., AP assassins or carry builds). Most hover below 20%. This skew justifies transforming `damageshare` using a quantile or log-based method during modeling to reduce outlier impact.


### Bivariate Analysis
For bivariate analysis, I looked at how objective control counts relate to win rate. Overall, there were positive correlations between team objective counts and the likelihood of winning. However, some relationships had unexpected shapes.

- Team Dragons vs. Win Rate

<iframe
 src="/LoL_junglers_performance_analysis/assets/team_dragons_vs_winrate.html"
 width="800"
 height="600"
 frameborder="0"
></iframe>

**Interpretation**: There is a clear positive relationship — teams that secure more dragons win more often, which reinforces the strategic value of dragon control.

- Team Barons vs. Win Rate

<iframe
 src="/LoL_junglers_performance_analysis/assets/team_barons_vs_winrate.html"
 width="800"
 height="600"
 frameborder="0"
></iframe>

**Interpretation**: The shape is bimodal; teams often either win without any barons or dominate after securing one or more, suggesting that Baron may act more as a win accelerator than a deciding factor.

- Opp. Team Dragons vs. Win Rate

<iframe
 src="/LoL_junglers_performance_analysis/assets/opp_team_dragons_vs_winrate.html"
 width="800"
 height="600"
 frameborder="0"
></iframe>

### Interesting Aggregations
An interesting aggregation is by patch and champion. This lets us observe how Jungler performance varies across different champions and across changing environments (patch versions).

<iframe src="/LoL_junglers_performance_analysis/assets/agg_patch_champion.html" width="800" height="400" frameborder="0">
</iframe>

**Significance**: This aggregation helps contextualize Jungler performance. If certain champions or patches lead to systematically better or worse stats, it suggests that win/loss outcomes may be influenced as much by the meta as by the individual player. In other words, Junglers might be blamed for poor outcomes that were more about champion viability or patch balance than their own decisions.

### Imputation
Zero imputations were necessary!!


---

## 📊 3. Feature Exploration <a name="feature-exploration"></a>

### Stating the Problem  
As mentioned in the introduction, the goal of this project is to determine whether Junglers are truly to blame for their team’s loss. To examine this, I framed the problem as a **binary classification task**: given a Jungler's performance stats in a match, can we predict whether their team won or lost?

### Target Variable  
The target is `result`, where:
- `1` = win  
- `0` = loss  

This makes it a clean 0/1 classification setup.

### Feature Selection  
To avoid confounding the model with non-Jungler factors, I used only **Jungler-centric or Jungler-related team objective features**:

- `earnedgoldshare`, `damageshare`, `kill_participation`, `cs_participation`, `deathshare`
- `team_dragons`, `team_barons`, `team_towers`, `team_inhibitors`
- `opp_team_dragons`, `opp_team_barons`, `opp_team_towers`, `opp_team_inhibitors`

These features capture both individual contribution and team-level performance signals often associated with Jungler impact (e.g., objective control, map pressure, survivability).

### Why This Problem Framing Works  
If a model trained only on Jungler-related stats can predict game outcomes with reasonable accuracy, then it suggests that **Junglers do have a measurable and consistent effect** on whether a team wins or loses. If the model performs poorly, it would suggest that outcomes are either:
- Driven by other roles or external factors (draft, coordination, etc.), or  
- Too noisy to reliably attribute to one role.

This approach allows for a statistical investigation of the blame narrative, grounded in real match data.

---

## 🤖 4. Baseline Model <a name="baseline-model"></a>

### Feature Selection  
For the baseline model, I used four core Jungler-related features:  
- `kill_participation`  
- `deathshare`  
- `earnedgoldshare`  
- `damageshare`  

These are metrics that are most relative to the other roles.

### Model Setup  
For baseline, I selected `Logistic Regression` (because why not) and `StandardScaler` for basic normalization. The model was evaluated on a held-out test set (20% split from the full dataset).

### Results  
- **Accuracy**: 0.531  
- **Precision (loss)**: 0.53  
- **Precision (win)**: 0.53  
- **Recall (loss)**: 0.50  
- **Recall (win)**: 0.57  

### Interpretation  
The baseline model performs only slightly better than random guessing. While Jungler stats like kill participation and gold share show some signal, they are not strong enough on their own to reliably predict game outcomes. This suggests that Junglers may influence wins or losses, but these features alone are not sufficient to “blame” them — or at least not to predict results with high confidence. This gives a good foundation to build a more sophisticated model using additional features and transformations in the next step.

---

## 🔧 5. Final Model <a name="final-model"></a>

### Feature Engineering
To improve on the baseline model, I added team-level objective features in addition to the original Jungler metrics. I then applied the following transformations using `ColumnTransformer`:

- **Standard-scaled features**:
  - `earnedgoldshare`, `damageshare`, `kill_participation`, `cs_participation`
- **Quantile-transformed features** (to normalize skew):
  - `deathshare`, `team_dragons`, `team_barons`, `team_towers`, `team_inhibitors`
  - `opp_team_dragons`, `opp_team_barons`, `opp_team_towers`, `opp_team_inhibitors`

These transformations helped ensure that both normally distributed and skewed features were standardized for effective model training.

### Model Tuning & Selection
I evaluated three classifiers using `GridSearchCV` with 5-fold cross-validation:

- **LogisticRegression**: `C = [0.1, 1, 10]`
- **SVC**: `C = [0.1, 1, 10]`, kernels = `['linear', 'rbf']`
- **RandomForestClassifier**: `n_estimators = [50, 100]`, `max_depth = [5, 10, None]`

All models used the same preprocessing pipeline and training/testing split.

### Best Model & Results

- **Best Model**: `SVC` with `C = 10`, `kernel = 'rbf'`
- **Test Accuracy**: **0.9828**

#### SVC Classification Report:
- **Precision (loss)**: 0.99  
- **Recall (loss)**: 0.98  
- **Precision (win)**: 0.98  
- **Recall (win)**: 0.99  
- **F1-score (macro avg)**: 0.98  

#### RandomForest
- **Accuracy**: 0.9815
- **Best Parameters**: `n_estimators=100`, `max_depth=10`
- **F1-score (macro avg)**: 0.98

#### Logistic Regression
- **Accuracy**: 0.9760
- **Best Parameters**: `C=10`
- **F1-score (macro avg)**: 0.98

### Confusion Matrix

Below are the confusion matrices for each final model, showing how often predictions matched the actual outcomes.

- **Logistic Regression**

<iframe src="/LoL_junglers_performance_analysis/assets/confusion_matrix_logisticregression.html" width="600" height="500" frameborder="0"></iframe>

- **Random Forest**

<iframe src="/LoL_junglers_performance_analysis/assets/confusion_matrix_randomforest.html" width="600" height="500" frameborder="0"></iframe>

- **Support Vector Classifier (SVC)**

<iframe src="/LoL_junglers_performance_analysis/assets/confusion_matrix_svc.html" width="600" height="500" frameborder="0"></iframe>

### Interpretation
All three models performed significantly better than the original baseline, with SVC achieving the highest accuracy. These results show that **Jungler performance, when properly contextualized with team objectives and transformed for skew, is highly predictive of match outcome**. In this dataset, Jungler-related stats were enough to predict wins and losses with nearly 99% accuracy — suggesting that the role's impact is not just perception, but measurable.

While this doesn't mean Junglers are *always* to blame, it strongly supports the idea that their performance is deeply tied to team success or failure.

---

