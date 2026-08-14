---
name: ml-model
description: Build an ML model end-to-end. Data prep → feature engineering → train → evaluate → deploy. Start simple, add complexity only when justified.
disable-model-invocation: false
allowed-tools: Task, Bash, Read, Write, Edit, Glob, Grep
---

# Model: $ARGUMENTS

## Step 1: Define the Problem

Before any code:

1. **Task type**: Classification, regression, ranking, clustering?
2. **Target variable**: What exactly are we predicting? How is it defined?
3. **Success metric**: What metric matters? (precision, recall, AUC, RMSE, NDCG?)
4. **Baseline**: What's the naive/human performance? (random guess, simple heuristic, current manual process)
5. **Data available**: What features exist? How much labeled data?
6. **How it's used**: Batch scoring, real-time, human-in-the-loop?

Present problem definition. Wait for confirmation.

## Step 2: Start with a Heuristic

Before ML, build a rule-based baseline:
```
If [simple condition] → predict [class/value]
```

For seller signal scoring example:
```
score = 0
if loan_maturity < 12_months: score += 3
if hold_period > 10_years: score += 2
if out_of_state_owner: score += 2
if vacancy > 30%: score += 2
if code_violations > 0: score += 1
```

This is your baseline to beat. If rules get you 80% of the way, you may not need ML yet.

## Step 3: Prepare Data

```
Raw Data → Analysis → Clean → Features → Train/Test Split
```

Data analysis (do this FIRST):
- Row count, column types, missing value rates
- Target distribution (balanced? skewed?)
- Feature distributions and outliers
- Correlations between features and target

Cleaning rules:
- Document every cleaning decision (why rows were dropped, how nulls were filled)
- Don't impute blindly — understand why data is missing
- Keep raw data intact, create cleaned copy

Train/test split:
- Time-based split for temporal data (train on past, test on future)
- NEVER use future data to predict past
- Hold out a true test set that you only touch once
- Use cross-validation on training set for model selection

## Step 4: Feature Engineering

Good features for CRE signal models:
- **Temporal**: Days since last sale, months to loan maturity, years held
- **Relative**: Price vs market median, vacancy vs submarket average
- **Categorical**: Property type, submarket, owner type (individual/LLC/trust)
- **Behavioral**: Listing history, price changes, days on market
- **External**: Interest rates, unemployment, permit activity

Rules:
- Start with 10-20 features, not 200
- Each feature should have a business hypothesis ("owners who've held 10+ years are more likely to sell because...")
- Avoid leakage (features that encode the answer)
- Log-transform skewed distributions
- One-hot encode categoricals (or use a model that handles them natively)

## Step 5: Train Models

Start simple, add complexity only when justified:

```
1. Logistic Regression / Linear Regression (interpretable baseline)
2. Random Forest (handles non-linearity, feature interactions)
3. Gradient Boosted Trees (XGBoost/LightGBM — usually the winner)
4. Only go deeper if 1-3 aren't good enough
```

For each model:
- Cross-validate on training set (5-fold)
- Track: metric, training time, feature importance
- Compare against heuristic baseline

DO NOT:
- ❌ Jump straight to deep learning
- ❌ Tune hyperparameters before getting the right features
- ❌ Use a model you can't explain to stakeholders
- ❌ Optimize a metric that doesn't match the business goal

## Step 6: Evaluate

On the held-out test set (ONE time):

For classification:
- Confusion matrix (understand the errors)
- Precision/Recall at different thresholds
- AUC-ROC and AUC-PR curves
- Calibration plot (are probabilities meaningful?)

For regression:
- RMSE, MAE, R²
- Residual plots (look for patterns in errors)
- Error distribution by segment

Key question: **Is the model better than the heuristic? By how much? Is the complexity worth it?**

## Step 7: Error Analysis

Look at the mistakes:
- What does the model get wrong? Is there a pattern?
- Are errors concentrated in a segment? (property type, geography, price range)
- What features would fix the errors? (this drives iteration)
- Are there data quality issues hiding in the errors?

This is more valuable than hyperparameter tuning.

## Step 8: Deploy (Keep It Simple)

For batch scoring (most CRE use cases):
```
1. Save model artifact (joblib/pickle + version tag)
2. Scoring script: load model → load new data → score → save results
3. Schedule via cron or pipeline orchestrator
4. Log: predictions, confidence, model version, timestamp
```

For monitoring:
- Track prediction distribution over time (drift detection)
- Compare predictions vs outcomes when ground truth arrives
- Alert if prediction distribution shifts significantly
- Retrain on a schedule or when performance degrades

## Step 9: Review & Commit

Spawn reviewer agent. Key review points:
- No data leakage
- Train/test split is valid (temporal if needed)
- Metrics are appropriate for the problem
- Model is interpretable enough for the use case
- Reproducibility: random seeds set, data versioned

Commit: `feat(model): add [target] [model_type] model`

## File Structure Convention

```
models/
  [model_name]/
    data_prep.py       # Cleaning + feature engineering
    train.py           # Model training + cross-validation
    evaluate.py        # Metrics + error analysis
    predict.py         # Batch scoring script
    config.py          # Hyperparameters, feature lists, thresholds
    artifacts/         # Saved models (gitignored if large)
    notebooks/         # EDA and analysis (optional)
    tests/
      test_features.py
      test_predict.py
```
