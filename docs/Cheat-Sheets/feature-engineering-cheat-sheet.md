# Feature Engineering Cheat Sheet

Raw data almost never arrives model-ready. Feature engineering is the process of transforming that raw data into representations a model can use effectively. The choices here — how you encode, scale, impute, and construct features — matter more than model selection in most real-world problems.

---

## 1. Numeric Transformations

Apply transformations to change the shape of a distribution. Most linear models and distance-based models (KNN, SVM) assume roughly symmetric, bounded input. Tree-based models are immune to monotonic transformations, but transformations can still help by stabilizing variance.

### log1p — Right-Skewed Data

Use when a numeric column has a long right tail (e.g., income, house prices, transaction amounts). `log1p` handles zeros; `log` does not.

```python
import numpy as np
import pandas as pd

df = pd.DataFrame({"revenue": [0, 100, 500, 2000, 150000]})

df["revenue_log"] = np.log1p(df["revenue"])

print(df)
# Output:
#    revenue  revenue_log
# 0        0     0.000000
# 1      100     4.615121
# 2      500     6.215606
# 3     2000     7.601402
# 4   150000    11.918391
```

### sqrt — Moderate Right Skew, Count Data

Gentler than log. Use for count variables (number of purchases, page views) where zeros are common and the tail is not extreme.

```python
df["page_views_sqrt"] = np.sqrt(df["page_views"])
```

### Box-Cox — Optimal Power Transform (No Zeros)

Finds the lambda that makes the distribution most Gaussian. Requires all positive values.

```python
from scipy.stats import boxcox

prices = df["price"].values  # must be strictly positive
transformed, fitted_lambda = boxcox(prices)

df["price_boxcox"] = transformed
print(f"Fitted lambda: {fitted_lambda:.4f}")
# Output: Fitted lambda: 0.2312  (close to 0 means log-like)
```

### Yeo-Johnson — Power Transform With Zeros and Negatives

Same idea as Box-Cox but supports zero and negative values. Prefer this in practice.

```python
from sklearn.preprocessing import PowerTransformer

pt = PowerTransformer(method="yeo-johnson")
df["revenue_yj"] = pt.fit_transform(df[["revenue"]])
```

### Winsorizing — Cap Outliers at Percentile Bounds

Use when outliers are measurement errors or rare events that distort model training but you do not want to drop rows. Clips values to a chosen percentile rather than removing them.

```python
lower = df["age"].quantile(0.01)
upper = df["age"].quantile(0.99)

df["age_winsorized"] = df["age"].clip(lower=lower, upper=upper)
```

---

## 2. Binning

Convert a continuous variable into discrete buckets. Use when you suspect a non-linear, step-function relationship between a variable and the target, or when the variable is noisy and the exact value matters less than the range.

### pd.cut — Equal-Width Bins

Each bin covers the same numeric range. Use when the distribution is uniform or you want intuitive, human-readable thresholds.

```python
df["age_band"] = pd.cut(
    df["age"],
    bins=[0, 18, 35, 50, 65, 100],
    labels=["<18", "18-35", "35-50", "50-65", "65+"],
    right=True
)
# Output: categorical column with ordered labels
```

### pd.qcut — Quantile (Equal-Frequency) Bins

Each bin contains roughly the same number of rows. Use when the distribution is skewed and you want balanced buckets.

```python
df["income_quartile"] = pd.qcut(
    df["income"],
    q=4,
    labels=["Q1", "Q2", "Q3", "Q4"],
    duplicates="drop"
)
```

### Custom Bins With Business Logic

Use when domain knowledge defines meaningful thresholds (e.g., credit score bands, BMI categories, regulatory buckets).

```python
credit_bins = [300, 580, 670, 740, 800, 850]
credit_labels = ["Poor", "Fair", "Good", "Very Good", "Exceptional"]

df["credit_tier"] = pd.cut(
    df["credit_score"],
    bins=credit_bins,
    labels=credit_labels
)
```

### When to Bin vs. Keep Continuous

Tree models find their own splits — binning adds no benefit and may lose information. Bin when:
- Feeding a linear or logistic regression where you suspect a step-function relationship
- The raw variable is too noisy (age rounded to nearest decade)
- You need human-interpretable features for reporting

> [!warning]
> Quantile bins computed on the full dataset before a train-test split will leak distributional information into the test set. Always compute bin edges on the training set and apply them to the test set.

---

## 3. Scaling

Most distance-based and gradient-based models are sensitive to feature magnitude. Tree models (Random Forest, XGBoost, LightGBM) are not — skip scaling for those.

### StandardScaler — Zero Mean, Unit Variance

Use as the default for linear models (logistic regression, linear SVM, ridge) and neural networks when data is roughly Gaussian.

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
df[["age_scaled", "income_scaled"]] = scaler.fit_transform(df[["age", "income"]])

# Mean ≈ 0, Std ≈ 1
print(df["age_scaled"].describe())
```

### MinMaxScaler — Compress to [0, 1]

Use when you need all values in a bounded range: KNN (so no feature dominates distance), neural networks with sigmoid outputs, or when the algorithm requires non-negative inputs.

```python
from sklearn.preprocessing import MinMaxScaler

mm_scaler = MinMaxScaler()
df["price_scaled"] = mm_scaler.fit_transform(df[["price"]])
```

### RobustScaler — Median and IQR

Use when the column has significant outliers that would distort the mean and standard deviation. Scales using the median and interquartile range, making it resistant to outliers.

```python
from sklearn.preprocessing import RobustScaler

rb_scaler = RobustScaler(quantile_range=(25.0, 75.0))
df["salary_scaled"] = rb_scaler.fit_transform(df[["salary"]])
```

> [!warning]
> Always fit scalers on the training set only. Fitting on the full dataset leaks test-set statistics (mean, min, max) into training, producing optimistically biased validation metrics.

---

## 4. Encoding Categoricals

Models need numbers. How you convert categories to numbers depends on: cardinality, whether the category is ordinal, and whether you have enough data per category.

### Label Encoding — Ordinal Categories

Use only when the category has a natural order (e.g., Low/Medium/High, Small/Medium/Large). Assigns integers that preserve rank.

```python
from sklearn.preprocessing import OrdinalEncoder

size_order = [["Small", "Medium", "Large", "XLarge"]]
enc = OrdinalEncoder(categories=size_order)

df["size_encoded"] = enc.fit_transform(df[["size"]])
# Output: Small→0, Medium→1, Large→2, XLarge→3
```

### One-Hot Encoding — Low-Cardinality Nominals

Use for unordered categories with fewer than ~15–20 unique values. Produces one binary column per category.

```python
df_encoded = pd.get_dummies(df, columns=["city"], drop_first=True, dtype=int)
# drop_first=True avoids multicollinearity (dummy variable trap)
```

### Target Encoding — High-Cardinality Nominals

Replace each category with the mean of the target variable for that category. Effective for high-cardinality columns (postal codes, product IDs, browser user-agent strings).

```python
# Compute on training set only
target_means = train_df.groupby("neighborhood")["price"].mean()
df["neighborhood_target_enc"] = df["neighborhood"].map(target_means)

# Unseen categories get NaN — fill with global mean
global_mean = train_df["price"].mean()
df["neighborhood_target_enc"].fillna(global_mean, inplace=True)
```

> [!warning]
> Target encoding computed on the full dataset causes severe target leakage. Use cross-validated target encoding in practice (available in `category_encoders.TargetEncoder` with `smoothing`).

### Frequency Encoding — Category → Count Proportion

Replace each category with how often it appears in the dataset. A lightweight proxy for target encoding when you want to avoid leakage risk or when the category's frequency is itself predictive.

```python
freq_map = df["device_type"].value_counts(normalize=True)
df["device_freq_enc"] = df["device_type"].map(freq_map)
```

---

## 5. Handling High Cardinality

A categorical column with thousands of unique values (ZIP codes, user IDs, product SKUs) is expensive to one-hot encode and too noisy for raw label encoding. Use one of these strategies.

### Group Rare Categories

Categories with very few observations are unreliable — collapse them into an "Other" bucket before encoding.

```python
min_count = 50  # threshold chosen based on training set size
freq = df["country"].value_counts()
rare_categories = freq[freq < min_count].index

df["country_grouped"] = df["country"].where(
    ~df["country"].isin(rare_categories), other="Other"
)
```

### Target Encoding With Smoothing

Smoothing pulls low-count estimates toward the global mean, reducing noise from rare categories.

```python
from category_encoders import TargetEncoder

enc = TargetEncoder(smoothing=10, min_samples_leaf=5)
df["postal_encoded"] = enc.fit_transform(df["postal_code"], df["churn"])
```

### Hashing Trick

Map categories to a fixed-size integer array using a hash function. Useful when you cannot hold all category counts in memory (streaming data, very large catalogs).

```python
from sklearn.feature_extraction import FeatureHasher

hasher = FeatureHasher(n_features=64, input_type="string")
hashed = hasher.transform(df["product_id"].apply(lambda x: [x]))
# Returns a sparse matrix with 64 columns
```

### Frequency Encoding (Revisited for High Cardinality)

Works well as a standalone feature — captures the model's implicit assumption that rare categories behave differently from common ones.

```python
user_freq = train_df["user_id"].value_counts().to_dict()
df["user_frequency"] = df["user_id"].map(user_freq).fillna(0)
```

---

## 6. Missing Value Strategies

Every strategy makes an assumption. Choose the one whose assumption matches what you know about why data is missing.

### Mean/Median Imputation

Use for numeric columns when missingness is random (MCAR) and you need a fast baseline. Use median when the column is skewed; it is more robust to outliers.

```python
from sklearn.impute import SimpleImputer

imputer = SimpleImputer(strategy="median")
df[["age", "income"]] = imputer.fit_transform(df[["age", "income"]])
```

### Missing Indicator Variable

When missingness itself is predictive (e.g., "no recorded income" may signal unemployment), add a binary flag before imputing. This preserves the signal.

```python
df["income_missing"] = df["income"].isna().astype(int)
df["income"] = df["income"].fillna(df["income"].median())
```

### KNN Imputation

Imputes each missing value using the mean of the k nearest complete neighbors in feature space. Better than mean imputation when features are correlated.

```python
from sklearn.impute import KNNImputer

knn_imp = KNNImputer(n_neighbors=5, weights="distance")
df_imputed = pd.DataFrame(
    knn_imp.fit_transform(df[["age", "bmi", "income"]]),
    columns=["age", "bmi", "income"]
)
```

### Iterative Imputation (Model-Based)

Treats each column with missing values as a regression target and uses the other columns as predictors. Iterates until convergence. Best when missing rates are high and features are highly correlated.

```python
from sklearn.experimental import enable_iterative_imputer  # noqa
from sklearn.impute import IterativeImputer
from sklearn.ensemble import RandomForestRegressor

iter_imp = IterativeImputer(
    estimator=RandomForestRegressor(n_estimators=50, random_state=42),
    max_iter=10,
    random_state=42
)
df_complete = pd.DataFrame(
    iter_imp.fit_transform(df[numeric_cols]),
    columns=numeric_cols
)
```

> [!warning]
> Fit all imputers on the training set only. Imputing before the split allows test-set values to influence training-set fill values.

---

## 7. DateTime Features

A raw timestamp is unusable as-is. Extract components that encode the patterns you expect: weekly seasonality, daily cycles, time elapsed since an event.

### Extract Calendar Components

```python
df["created_at"] = pd.to_datetime(df["created_at"])

df["year"]        = df["created_at"].dt.year
df["month"]       = df["created_at"].dt.month          # 1–12
df["day"]         = df["created_at"].dt.day
df["hour"]        = df["created_at"].dt.hour
df["day_of_week"] = df["created_at"].dt.dayofweek      # 0=Monday, 6=Sunday
df["is_weekend"]  = (df["day_of_week"] >= 5).astype(int)
df["quarter"]     = df["created_at"].dt.quarter
```

### Days Since a Reference Event

Use when the time elapsed since a specific event is more predictive than the absolute date (e.g., days since last purchase, days since account creation).

```python
reference_date = pd.Timestamp("2024-01-01")
df["days_since_signup"] = (df["created_at"] - reference_date).dt.days
```

### Cyclical Encoding — Preserve Periodicity

Hour 23 and hour 0 are adjacent but numerically far apart. Encode cyclical features as sin/cos pairs so the model learns this adjacency.

```python
import numpy as np

# Hour of day — period 24
df["hour_sin"] = np.sin(2 * np.pi * df["hour"] / 24)
df["hour_cos"] = np.cos(2 * np.pi * df["hour"] / 24)

# Month of year — period 12
df["month_sin"] = np.sin(2 * np.pi * df["month"] / 12)
df["month_cos"] = np.cos(2 * np.pi * df["month"] / 12)

# Day of week — period 7
df["dow_sin"] = np.sin(2 * np.pi * df["day_of_week"] / 7)
df["dow_cos"] = np.cos(2 * np.pi * df["day_of_week"] / 7)
```

### Business Day Flags

```python
import numpy as np

df["is_month_start"] = df["created_at"].dt.is_month_start.astype(int)
df["is_month_end"]   = df["created_at"].dt.is_month_end.astype(int)
df["is_quarter_end"] = df["created_at"].dt.is_quarter_end.astype(int)
```

---

## 8. Text Features

Text must be converted to numeric representations. The right representation depends on vocabulary size, model type, and whether word order matters.

### Word Count and Character Features

Use as lightweight numeric signals before building heavier NLP features. Often surprisingly predictive in classification tasks.

```python
df["word_count"]       = df["review_text"].str.split().str.len()
df["char_count"]       = df["review_text"].str.len()
df["avg_word_length"]  = df["char_count"] / (df["word_count"] + 1)
df["sentence_count"]   = df["review_text"].str.count(r"[.!?]+")
df["exclamation_count"] = df["review_text"].str.count("!")
```

### Bag of Words

Each unique word becomes a column. Cell value = word count in that document. High-dimensional and sparse. Use for short texts and simple classifiers.

```python
from sklearn.feature_extraction.text import CountVectorizer

bow = CountVectorizer(max_features=5000, min_df=5, stop_words="english")
bow_matrix = bow.fit_transform(df["review_text"])
# Returns a sparse matrix of shape (n_samples, 5000)
```

### TF-IDF

Weights words by how often they appear in a document relative to how common they are across all documents. Rare but frequent words in a document get high scores; common stopwords get low scores.

```python
from sklearn.feature_extraction.text import TfidfVectorizer

tfidf = TfidfVectorizer(
    max_features=10000,
    ngram_range=(1, 2),   # unigrams + bigrams
    min_df=3,
    sublinear_tf=True     # apply log to term frequency
)
tfidf_matrix = tfidf.fit_transform(train_df["review_text"])
```

### Character N-Grams

Captures morphological patterns (prefixes, suffixes, misspellings). Useful for noisy user-generated text, domain-specific jargon, or non-English text.

```python
char_tfidf = TfidfVectorizer(
    analyzer="char_wb",   # character n-grams, padded with word boundaries
    ngram_range=(3, 5),
    max_features=20000,
    min_df=5
)
char_matrix = char_tfidf.fit_transform(df["product_name"])
```

---

## 9. Interaction Features

When the relationship between two features and the target is not independent, a model may learn this interaction on its own (trees) or may need explicit help (linear models). Build interactions deliberately from domain knowledge before resorting to brute-force combinations.

### Multiplicative Interaction

Use when you believe two features amplify each other's effect (e.g., area × price_per_sqft, or hours_worked × hourly_rate).

```python
df["total_cost"] = df["unit_price"] * df["quantity"]
df["area_sqft"]  = df["length_ft"] * df["width_ft"]
```

### Ratios

Normalize one measurement by another. Ratios often capture relationships that neither raw variable expresses alone.

```python
df["debt_to_income"]    = df["total_debt"] / (df["annual_income"] + 1)
df["revenue_per_user"]  = df["monthly_revenue"] / (df["active_users"] + 1)
df["expense_ratio"]     = df["operating_expenses"] / (df["revenue"] + 1)
```

### Polynomial Features

Automatically generates all degree-2 combinations: squares and pairwise products. Use for linear models when you suspect curved relationships. Cardinality grows as O(n²) — use with at most 10–20 input features.

```python
from sklearn.preprocessing import PolynomialFeatures

poly = PolynomialFeatures(degree=2, interaction_only=False, include_bias=False)
numeric_cols = ["age", "income", "credit_score"]

poly_features = poly.fit_transform(df[numeric_cols])
poly_df = pd.DataFrame(poly_features, columns=poly.get_feature_names_out(numeric_cols))
```

### Domain-Driven Combinations

Build features that a practitioner in the field would consider meaningful. These tend to generalize better than brute-force combinations.

```python
# E-commerce: user's average order value
df["avg_order_value"] = df["total_spend"] / (df["order_count"] + 1)

# Credit: utilization rate
df["credit_utilization"] = df["current_balance"] / (df["credit_limit"] + 1)

# Healthcare: BMI from height and weight
df["bmi"] = df["weight_kg"] / (df["height_m"] ** 2)
```

---

## 10. Aggregation Features

Summarize groups to add context at a higher level of granularity. A customer's raw transaction history is less useful than aggregate statistics about that customer. Essential for relational and time-series data.

### GroupBy Statistics

For each row, look up aggregate statistics of the group it belongs to. Adds global context to each observation.

```python
# Mean and std of price by product category
category_stats = train_df.groupby("category")["price"].agg(
    category_mean_price="mean",
    category_std_price="std",
    category_count="count"
).reset_index()

df = df.merge(category_stats, on="category", how="left")

# Price relative to category mean
df["price_vs_category_mean"] = df["price"] / (df["category_mean_price"] + 1)
```

### User-Level Aggregation

Aggregate a user's history into a single row of features before joining to the main table.

```python
user_features = transactions_df.groupby("user_id").agg(
    total_spend=("amount", "sum"),
    num_transactions=("amount", "count"),
    avg_transaction=("amount", "mean"),
    max_transaction=("amount", "max"),
    days_since_first=("transaction_date", lambda x: (pd.Timestamp.today() - x.min()).days),
    days_since_last=("transaction_date", lambda x: (pd.Timestamp.today() - x.max()).days)
).reset_index()

df = df.merge(user_features, on="user_id", how="left")
```

### Rolling Window Features for Time Series

For each timestamp, aggregate the previous N periods. Captures trend and momentum without leaking future information.

```python
# Sort by entity and time first
df = df.sort_values(["store_id", "date"])

df["sales_7d_avg"]    = (df.groupby("store_id")["daily_sales"]
                           .transform(lambda x: x.rolling(7, min_periods=1).mean()))

df["sales_7d_std"]    = (df.groupby("store_id")["daily_sales"]
                           .transform(lambda x: x.rolling(7, min_periods=1).std()))

df["sales_28d_avg"]   = (df.groupby("store_id")["daily_sales"]
                           .transform(lambda x: x.rolling(28, min_periods=1).mean()))

# Lag features — yesterday's sales as a predictor of today's
df["sales_lag_1"]     = df.groupby("store_id")["daily_sales"].shift(1)
df["sales_lag_7"]     = df.groupby("store_id")["daily_sales"].shift(7)
```

> [!warning]
> Rolling and lag features computed without grouping by entity will bleed one store's data into another store's features. Always group by the entity identifier before rolling.

---

## 11. Feature Selection

More features are not always better. Irrelevant features add noise, increase training time, risk overfitting, and make models harder to interpret. Trim aggressively after generating candidates.

### Variance Threshold — Remove Near-Constant Features

A feature with near-zero variance carries almost no information. Remove it before modeling.

```python
from sklearn.feature_selection import VarianceThreshold

selector = VarianceThreshold(threshold=0.01)  # removes features with variance < 0.01
X_reduced = selector.fit_transform(X_train)

kept_cols = X_train.columns[selector.get_support()].tolist()
print(f"Kept {len(kept_cols)} of {X_train.shape[1]} features")
```

### Correlation Filter — Remove Redundant Features

Two highly correlated features carry the same information. Keep one; drop the other. Use a threshold of 0.90–0.95 for a conservative filter.

```python
corr_matrix = X_train.corr().abs()
upper_triangle = corr_matrix.where(
    np.triu(np.ones(corr_matrix.shape), k=1).astype(bool)
)
to_drop = [col for col in upper_triangle.columns if any(upper_triangle[col] > 0.95)]
X_train_filtered = X_train.drop(columns=to_drop)
print(f"Dropped {len(to_drop)} correlated features: {to_drop}")
```

### VIF — Multicollinearity Detection for Linear Models

Variance Inflation Factor measures how much a feature's variance inflates due to linear correlation with other features. VIF > 5–10 indicates problematic multicollinearity. Relevant only for linear/logistic regression.

```python
from statsmodels.stats.outliers_influence import variance_inflation_factor

def compute_vif(X: pd.DataFrame) -> pd.DataFrame:
    vif_data = pd.DataFrame({
        "feature": X.columns,
        "VIF": [variance_inflation_factor(X.values, i) for i in range(X.shape[1])]
    })
    return vif_data.sort_values("VIF", ascending=False)

vif_df = compute_vif(X_train[numeric_features])
print(vif_df[vif_df["VIF"] > 5])
```

### Recursive Feature Elimination

Trains a model, ranks features by importance, removes the weakest, and repeats. More expensive but often finds a better subset than filter methods.

```python
from sklearn.feature_selection import RFECV
from sklearn.ensemble import RandomForestClassifier

estimator = RandomForestClassifier(n_estimators=100, random_state=42)
rfecv = RFECV(
    estimator=estimator,
    step=1,
    cv=5,
    scoring="roc_auc",
    min_features_to_select=5,
    n_jobs=-1
)
rfecv.fit(X_train, y_train)

selected_features = X_train.columns[rfecv.support_].tolist()
print(f"Optimal feature count: {rfecv.n_features_}")
print(f"Selected features: {selected_features}")
```

### Tree-Based Feature Importance

A fast heuristic for shortlisting features. Use as a starting point, not a final answer — importance is biased toward high-cardinality and correlated features.

```python
import pandas as pd
from sklearn.ensemble import RandomForestClassifier

model = RandomForestClassifier(n_estimators=200, random_state=42)
model.fit(X_train, y_train)

importance_df = pd.DataFrame({
    "feature": X_train.columns,
    "importance": model.feature_importances_
}).sort_values("importance", ascending=False)

# Keep top 30 features
top_features = importance_df.head(30)["feature"].tolist()
```

---

## 12. Leakage Prevention

Data leakage is when information from outside the training distribution enters the model during training. It inflates validation metrics and produces models that fail in production. It is the single most common cause of the "works in notebook, fails in production" problem.

### Always Fit on Train, Transform on Test

Every preprocessing step that learns from data (scalers, encoders, imputers) must be fit exclusively on the training set.

```python
from sklearn.preprocessing import StandardScaler

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)   # fit + transform
X_test_scaled  = scaler.transform(X_test)         # transform only — no fit
```

### Use Pipeline to Prevent Leakage by Design

`Pipeline` chains preprocessing and modeling steps so that `fit` on the pipeline only fits the preprocessor on training data. Cross-validation with a pipeline is leak-free.

```python
from sklearn.pipeline import Pipeline
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import cross_val_score

pipeline = Pipeline([
    ("impute",  SimpleImputer(strategy="median")),
    ("scale",   StandardScaler()),
    ("model",   LogisticRegression(max_iter=1000))
])

# cross_val_score refits the entire pipeline on each fold's training set
scores = cross_val_score(pipeline, X, y, cv=5, scoring="roc_auc")
print(f"CV AUC: {scores.mean():.4f} ± {scores.std():.4f}")
```

### Temporal Ordering — No Future Data in Time Series

When your data has a time dimension, never train on data from after the validation period. Split by time, not randomly.

```python
df = df.sort_values("date")

cutoff = pd.Timestamp("2024-01-01")
train_df = df[df["date"] < cutoff]
test_df  = df[df["date"] >= cutoff]

# Fit all encoders and scalers on train_df only
target_means = train_df.groupby("store_id")["sales"].mean()
test_df["store_target_enc"] = test_df["store_id"].map(target_means)
```

### Leakage Red Flags Checklist

```python
# Common leak patterns — check your feature set against these

# 1. Future information in a time-based problem
#    e.g., "total sales for this store" computed over the full dataset
#    Fix: compute rolling/historical aggregates from train-period data only

# 2. Target-derived features computed before the split
#    e.g., target encoding fitted on the full dataset
#    Fix: fit inside cross-validation folds or use smoothed leave-one-out encoding

# 3. Post-event features
#    e.g., "claim amount" as a feature when predicting "will file a claim"
#    Fix: remove any feature that is logically caused by the target

# 4. ID columns leaking through
#    e.g., sequential user_id where lower IDs correspond to a different time period
#    Fix: exclude IDs from features; they carry no causal signal

# 5. Preprocessing fitted on the full dataset before cross-validation
#    Fix: wrap all preprocessing in a Pipeline and use cross_val_score on the pipeline
```

> [!success]
> A simple rule: anything that requires knowledge of the full dataset (including the test set) to compute is a potential leak. Run every preprocessing step inside a Pipeline and validate with `cross_val_score` — not a manual train/test split — to catch leakage automatically.
