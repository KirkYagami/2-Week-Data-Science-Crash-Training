# Feature Engineering

Feature engineering is where domain expertise meets statistical thinking. Most experienced practitioners will tell you that good features matter more than model choice. These questions probe your ability to turn raw data into signal.

---

### Q1: What is feature engineering and why does it matter?

??? "Show answer"
    Feature engineering is the process of using domain knowledge and statistical intuition to create, transform, or select input features that make a learning algorithm work better. It bridges raw data and the model's ability to extract signal from it.

    **Why it matters:**
    - Most real-world data is not in a form that directly represents the underlying patterns. A model can only learn relationships that exist in the features you provide.
    - Tree-based models can approximate many transformations implicitly (they learn thresholds), but linear models and neural networks often need explicit feature construction.
    - Good features can turn a mediocre model into a strong one. Poor features will limit even the most sophisticated algorithm.

    **Examples of high-impact feature engineering:**
    - From a timestamp, extract hour of day, day of week, is_weekend, days_since_last_purchase.
    - Log-transform a right-skewed monetary variable to make it approximately normal.
    - Create a ratio feature: `revenue_per_user = total_revenue / active_users`.
    - From GPS coordinates, compute distance to the nearest store.

    The best feature engineers ask: "What information would a domain expert use to make this prediction, and how do I encode that information numerically?"

---

### Q2: What is the difference between feature selection and feature extraction?

??? "Show answer"
    **Feature selection** chooses a subset of the original features and discards the rest. The selected features retain their original meaning and interpretability.

    - Examples: removing low-variance features, selecting the top-k features by mutual information, Lasso (embedded selection).
    - Use when: interpretability matters, you want to understand which original variables are predictive, dataset has many irrelevant/noisy features.

    **Feature extraction** creates new features by transforming or combining the original features. The new features may not have a direct interpretation in terms of the original variables.

    - Examples: PCA (linear combinations of original features), t-SNE, autoencoders, TF-IDF from raw text, mel-spectrogram from audio.
    - Use when: features are high-dimensional and redundant (images, text, audio), you care more about predictive performance than interpretability.

    **A concrete comparison:**
    - You have 100 correlated demographic features. **Feature selection** might keep the 20 most informative ones. **Feature extraction** via PCA might produce 10 uncorrelated components that explain 95% of the variance.
    - For a downstream linear model, PCA components often produce better results. For a tree model, selection is usually preferable (trees handle correlated features reasonably well and selecting preserves interpretability).

---

### Q3: How do you handle missing values?

??? "Show answer"
    Missing data handling depends on why data is missing (missing mechanism) and what algorithm you're using.

    **Missing mechanisms:**
    - **MCAR (Missing Completely at Random)**: missingness is unrelated to any variable. Safe to drop or impute.
    - **MAR (Missing at Random)**: missingness depends on other observed variables (not the missing value itself). Imputation is appropriate.
    - **MNAR (Missing Not at Random)**: the missing value itself causes the missingness (e.g., high-earners skip the income field). Imputation is biased; consider the missingness as a feature.

    **Strategies:**

    1. **Drop rows**: only safe for MCAR with small missing fraction. Wastes data.
    2. **Drop columns**: when a column has >50–60% missing values and missingness isn't informative.
    3. **Simple imputation**: fill with mean/median (continuous) or mode (categorical).
    4. **Indicator + imputation**: add a binary `feature_is_missing` column, then impute. This preserves the signal from the missingness pattern.
    5. **Model-based imputation**: use KNN, random forest, or iterative imputation to predict the missing value from other features (more accurate but computationally expensive).
    6. **Native handling**: XGBoost, LightGBM, CatBoost learn the optimal direction for missing values during training — no imputation needed.

    ```python
    from sklearn.impute import SimpleImputer, KNNImputer
    from sklearn.experimental import enable_iterative_imputer
    from sklearn.impute import IterativeImputer
    import pandas as pd

    # Add missingness indicator before imputing
    df['age_missing'] = df['age'].isna().astype(int)

    # Simple median imputation
    imputer = SimpleImputer(strategy='median')
    df['age_imputed'] = imputer.fit_transform(df[['age']])

    # MICE (Multiple Imputation by Chained Equations)
    mice = IterativeImputer(random_state=42)
    X_imputed = mice.fit_transform(X)
    ```

---

### Q4: What is imputation and what are its risks?

??? "Show answer"
    Imputation replaces missing values with estimated values so that an algorithm that can't handle NaN can be applied.

    **Risks of imputation:**

    - **Bias**: imputed values are estimates, not ground truth. If the imputation model is wrong, it introduces systematic error. Mean imputation assumes the missing value is average — which is often false.
    - **Underestimation of uncertainty**: after imputation, a model treats imputed values as observed. Downstream uncertainty estimates (confidence intervals, p-values) are too narrow because the model doesn't account for the uncertainty in the imputed values.
    - **Data leakage if done before split**: fitting an imputer on the full dataset before splitting allows test-set information to influence training-set imputation. Always fit the imputer on training data only.

    ```python
    from sklearn.pipeline import Pipeline
    from sklearn.impute import SimpleImputer
    from sklearn.ensemble import GradientBoostingClassifier
    from sklearn.model_selection import cross_val_score

    # Correct: imputer is part of the pipeline, fit only on training fold
    pipe = Pipeline([
        ('imputer', SimpleImputer(strategy='median')),
        ('model', GradientBoostingClassifier())
    ])
    scores = cross_val_score(pipe, X, y, cv=5)  # no leakage
    ```

    - **Distortion of distributions and relationships**: mean imputation reduces variance and can distort correlations between features. Multiple imputation (MICE) is more statistically sound for inference, but adds complexity.

    The safest approach for production ML: add a binary missingness indicator, impute with a simple statistic, fit the imputer inside a pipeline, and let tree-based models handle the rest.

---

### Q5: What is one-hot encoding and when is it problematic?

??? "Show answer"
    One-hot encoding (OHE) converts a categorical variable with K unique categories into K binary columns, where exactly one column is 1 and the rest are 0.

    ```python
    import pandas as pd
    from sklearn.preprocessing import OneHotEncoder

    # pandas
    dummies = pd.get_dummies(df['colour'], drop_first=True)  # K-1 columns

    # sklearn (handles unseen categories)
    ohe = OneHotEncoder(handle_unknown='ignore', sparse_output=False, drop='first')
    encoded = ohe.fit_transform(df[['colour']])
    ```

    **When it works:** small number of categories (K < ~20), no ordinal relationship, linear models or neural networks.

    **When it's problematic:**

    - **High cardinality**: a feature with 10,000 unique values creates 10,000 sparse columns. The resulting matrix is enormous, slows training, and most columns will be nearly all zeros. Solution: target encoding, frequency encoding, or embedding.
    - **Multicollinearity (dummy variable trap)**: with K columns summing to 1, any one column is a linear combination of the others. For linear models, drop one column (`drop='first'`). Tree models are immune to this.
    - **Unseen categories at inference**: if the test set contains categories not seen during training, OHE throws an error or produces wrong output. Use `handle_unknown='ignore'` in sklearn.
    - **Memory and speed**: for very large datasets with many categorical columns, OHE can blow up memory usage.

---

### Q6: What is target encoding and what is its risk?

??? "Show answer"
    Target encoding replaces each category level with the mean (or other statistic) of the target variable for that level. For binary classification, it's the mean of the positive class probability.

    ```python
    # category_encoders library
    import category_encoders as ce

    te = ce.TargetEncoder(cols=['city'], smoothing=10)
    X_train_enc = te.fit_transform(X_train, y_train)
    X_test_enc = te.transform(X_test)  # uses training statistics only
    ```

    **Why it works:** captures the predictive relationship between the category and the target in a single numeric feature. Handles high-cardinality features efficiently (1 column regardless of K).

    **The risk: target leakage**

    If you compute target statistics on the full training set and then train on the same data, the model sees the target encoded into the feature — it's effectively using `y` to predict `y`. This inflates training performance and causes overfitting.

    **Proper implementation requires:**
    1. **Out-of-fold encoding**: for each training fold in cross-validation, compute target statistics from the other folds only. Sklearn's `TargetEncoder` and `category_encoders` handle this correctly.
    2. **Smoothing**: blend the category mean with the global mean to regularise estimates for rare categories: `encoded = (count × category_mean + m × global_mean) / (count + m)`. The smoothing parameter `m` controls how much to trust sparse categories.
    3. **Always fit on training data only** and transform test data using those training-set statistics.

---

### Q7: What is ordinal encoding and when is it appropriate?

??? "Show answer"
    Ordinal encoding assigns an integer to each category, preserving a meaningful rank order. For example: `{'Low': 0, 'Medium': 1, 'High': 2}`.

    ```python
    from sklearn.preprocessing import OrdinalEncoder

    oe = OrdinalEncoder(categories=[['Low', 'Medium', 'High']])
    X_enc = oe.fit_transform(df[['education_level']])
    ```

    **When it's appropriate:**
    - The categories have a genuine rank order that the model should respect (education level, survey rating scales, severity scores).
    - You're using tree-based models — trees find the right threshold regardless of the specific numeric values assigned, so ordinal encoding works fine even if the spacing is arbitrary.

    **When NOT to use it:**
    - For nominal categories (colours, city names, species) — ordinal encoding imposes a false ordering. `Red=0, Blue=1, Green=2` implies `Green > Blue > Red`, which is meaningless. Use OHE or target encoding instead.
    - For linear models with ordinal features where the spacing matters — the model assumes linear effects, so `Low=0, Medium=1, High=2` implies equal spacing between levels. If the actual effect is non-linear, encode differently or add interaction terms.

---

### Q8: How do you handle high-cardinality categorical features?

??? "Show answer"
    High-cardinality features (hundreds or thousands of unique values) make one-hot encoding impractical. Several strategies work depending on context:

    **1. Target encoding (with smoothing and OOF):** replace each category with the smoothed mean target value. Most effective when cardinality is high and the feature is genuinely predictive.

    **2. Frequency / count encoding:** replace each category with how often it appears in the training data. Captures popularity signals without target leakage risk.

    ```python
    freq_map = X_train['city'].value_counts().to_dict()
    X_train['city_freq'] = X_train['city'].map(freq_map)
    X_test['city_freq'] = X_test['city'].map(freq_map).fillna(0)
    ```

    **3. Hashing (feature hashing / hashing trick):** hash the category string into a fixed-size vector of size n_features using a hash function. Fast, constant memory, but causes hash collisions.

    ```python
    from sklearn.feature_extraction import FeatureHasher
    fh = FeatureHasher(n_features=256, input_type='string')
    ```

    **4. Grouping rare categories:** bin all categories with count < threshold into an "Other" bucket, then OHE the resulting reduced set.

    **5. Embeddings (deep learning):** for neural networks, learn a dense low-dimensional vector representation for each category. Captures semantic similarity. Standard in recommendation systems and NLP.

    **6. Native handling:** LightGBM and CatBoost handle high-cardinality categoricals natively — pass the column as categorical and let the algorithm handle encoding internally.

---

### Q9: What is feature scaling and which algorithms require it?

??? "Show answer"
    Feature scaling transforms numerical features to a common scale. Without it, features with larger numeric ranges dominate distance calculations and gradient steps, leading to poor model performance.

    **Algorithms that REQUIRE scaling:**
    - **KNN**: distance between points is dominated by high-magnitude features.
    - **SVM**: the kernel computes inner products; unscaled features bias the margin.
    - **Linear/logistic regression**: gradient descent converges much faster with scaled features. Coefficients also become comparable.
    - **Neural networks**: unscaled inputs cause vanishing/exploding gradients and slow convergence.
    - **PCA / clustering**: distance-based; unscaled features dominate the principal components and centroids.
    - **Lasso/Ridge**: regularisation penalises large coefficients — coefficients must be on the same scale for the penalty to be applied fairly.

    **Algorithms that do NOT require scaling:**
    - **Decision trees and tree ensembles (Random Forest, XGBoost, LightGBM)**: trees split on thresholds — the ordering of values matters, not their magnitude.
    - **Naive Bayes**: computes probabilities per feature independently.

    ```python
    from sklearn.preprocessing import StandardScaler, MinMaxScaler
    from sklearn.pipeline import Pipeline
    from sklearn.linear_model import LogisticRegression

    pipe = Pipeline([
        ('scaler', StandardScaler()),
        ('clf', LogisticRegression())
    ])
    # Always scale INSIDE the pipeline to prevent leakage
    ```

---

### Q10: What is the difference between normalisation and standardisation?

??? "Show answer"
    **Normalisation (Min-Max scaling):** scales each feature to a fixed range, typically [0, 1].

    ```
    x_scaled = (x - min(x)) / (max(x) - min(x))
    ```

    ```python
    from sklearn.preprocessing import MinMaxScaler
    scaler = MinMaxScaler(feature_range=(0, 1))
    ```

    - Output is bounded. Useful when you need values in a specific range (e.g., neural network input layers, image pixel values).
    - Sensitive to outliers: a single extreme value compresses all other values into a narrow range.

    **Standardisation (Z-score scaling):** transforms features to have zero mean and unit variance.

    ```
    x_scaled = (x - mean(x)) / std(x)
    ```

    ```python
    from sklearn.preprocessing import StandardScaler
    scaler = StandardScaler()
    ```

    - Output is not bounded (can be any value).
    - Robust to the range of the data.
    - Less sensitive to outliers than normalisation (though still not immune).
    - Assumes the feature is approximately Gaussian — the scaled version has mean 0, std 1, which matches what many algorithms (linear models, SVMs) implicitly assume.

    **Which to use:**
    - Default to **StandardScaler** for most ML tasks.
    - Use **MinMaxScaler** when you need bounded outputs (image processing, neural network inputs with specific activation functions).
    - Use **RobustScaler** (scales using median and IQR) when your data has significant outliers.

---

### Q11: What is PCA and how does it reduce dimensions?

??? "Show answer"
    Principal Component Analysis (PCA) finds a lower-dimensional linear subspace that captures the maximum variance in the data.

    **How it works:**
    1. Center the data (subtract the mean).
    2. Compute the covariance matrix `Σ`.
    3. Find the eigenvectors and eigenvalues of `Σ`. Eigenvectors are the principal components (directions of maximum variance); eigenvalues give the variance explained by each.
    4. Sort eigenvectors by descending eigenvalue.
    5. Project the data onto the top-k eigenvectors: `X_reduced = X · W_k`

    ```python
    from sklearn.decomposition import PCA
    from sklearn.preprocessing import StandardScaler

    # Always scale before PCA
    X_scaled = StandardScaler().fit_transform(X)

    pca = PCA(n_components=0.95)   # retain 95% of variance
    X_reduced = pca.fit_transform(X_scaled)

    print(pca.explained_variance_ratio_)  # variance explained by each component
    print(pca.n_components_)              # number of components chosen
    ```

    **What it actually does:** rotates the feature space so that the first axis points in the direction of maximum variance, the second axis (orthogonal to the first) points in the direction of second-maximum variance, and so on. By keeping only the top-k axes, you discard directions of low variance — typically noise.

    **Limitations:**
    - Only captures **linear** structure. Non-linear dimensionality reduction (t-SNE, UMAP, kernel PCA) is needed for manifold structure.
    - Components are not interpretable in terms of original features.
    - Requires scaling before application.

---

### Q12: How do you create features from datetime columns?

??? "Show answer"
    Datetime columns are among the richest sources of engineered features because time encodes numerous patterns — seasonality, trends, periodicity, and temporal context.

    ```python
    import pandas as pd

    df['timestamp'] = pd.to_datetime(df['timestamp'])

    # Calendar features
    df['hour']          = df['timestamp'].dt.hour
    df['day_of_week']   = df['timestamp'].dt.dayofweek     # 0=Monday
    df['day_of_month']  = df['timestamp'].dt.day
    df['week_of_year']  = df['timestamp'].dt.isocalendar().week.astype(int)
    df['month']         = df['timestamp'].dt.month
    df['quarter']       = df['timestamp'].dt.quarter
    df['year']          = df['timestamp'].dt.year
    df['is_weekend']    = df['day_of_week'].isin([5, 6]).astype(int)

    # Cyclical encoding for periodic features (preserves continuity)
    import numpy as np
    df['hour_sin'] = np.sin(2 * np.pi * df['hour'] / 24)
    df['hour_cos'] = np.cos(2 * np.pi * df['hour'] / 24)

    # Temporal context
    reference_date = pd.Timestamp('2020-01-01')
    df['days_since_ref'] = (df['timestamp'] - reference_date).dt.days

    # Lag features (for time series)
    df['prev_day_sales'] = df['sales'].shift(1)
    df['rolling_7d_avg'] = df['sales'].rolling(7).mean()
    ```

    **Cyclical encoding matters:** if you encode hour as a raw integer 0–23, the model sees hour 23 and hour 0 as far apart (23 units), even though they're consecutive. Sine/cosine encoding maps the cycle correctly, preserving the fact that midnight connects to the next day.

---

### Q13: What is feature interaction and how do you create interaction features?

??? "Show answer"
    A feature interaction captures the combined effect of two or more features — cases where the effect of one feature depends on the value of another. Linear models cannot learn interactions from raw features alone; they need interactions to be explicitly created.

    **Example:** age and income might individually be weak predictors of loan default, but their combination (age × income) might be strongly predictive. A young person with low income has a very different risk profile than an older person with the same income.

    **Creating interaction features:**

    ```python
    import pandas as pd
    from sklearn.preprocessing import PolynomialFeatures

    # Manual interaction
    df['age_x_income'] = df['age'] * df['income']
    df['income_per_dependent'] = df['income'] / (df['n_dependents'] + 1)

    # Automated pairwise interactions (sklearn)
    from sklearn.preprocessing import PolynomialFeatures
    poly = PolynomialFeatures(degree=2, interaction_only=True, include_bias=False)
    X_interactions = poly.fit_transform(X)
    # For 10 features: creates 10 + C(10,2) = 55 features
    ```

    **Tree-based models create interactions implicitly** through multi-level splits — a split on `age` followed by a split on `income` within that branch is equivalent to an interaction. This is one reason tree models are so powerful on tabular data without explicit interaction features.

    For linear models, adding interactions dramatically expands the feature space. Use domain knowledge to create only the interactions you believe are meaningful, or use regularisation to handle automatic interaction generation.

---

### Q14: What is binning and when is it useful?

??? "Show answer"
    Binning (discretisation) converts a continuous feature into discrete buckets. For example, age → {18-25, 26-35, 36-50, 51+}.

    ```python
    import pandas as pd
    from sklearn.preprocessing import KBinsDiscretizer

    # Custom bins
    df['age_group'] = pd.cut(df['age'],
                              bins=[0, 25, 35, 50, 100],
                              labels=['young', 'adult', 'middle_aged', 'senior'])

    # Quantile binning (equal frequency bins)
    df['income_quartile'] = pd.qcut(df['income'], q=4, labels=False)

    # sklearn
    kbd = KBinsDiscretizer(n_bins=5, encode='ordinal', strategy='quantile')
    X_binned = kbd.fit_transform(X[['age']])
    ```

    **When binning is useful:**
    - **Robustness to outliers**: extreme values fall into the top/bottom bin and don't distort the model.
    - **Capturing non-linear relationships in linear models**: after binning + OHE, a linear model can fit a different coefficient to each bin, effectively modelling a step function.
    - **Handling noise**: if precise values are noisy but the bin is reliable (e.g., "50–60 years old" is meaningful; the exact age has measurement error).
    - **Feature engineering intuition**: age brackets often have distinct behaviour that domain experts recognise.

    **When not to bin:**
    - For tree-based models: binning is unnecessary — trees find their own optimal thresholds. Manual binning can only lose information.
    - When continuous precision matters and you have enough data for the model to learn the relationship.

---

### Q15: How do you detect and handle outliers?

??? "Show answer"
    Outliers are data points that are far from the rest of the distribution. They can be genuine rare events (important to model correctly), measurement errors (should be fixed or removed), or data entry errors.

    **Detection methods:**

    ```python
    import numpy as np
    import pandas as pd

    # Z-score method (parametric)
    z_scores = (df['value'] - df['value'].mean()) / df['value'].std()
    outliers_z = df[np.abs(z_scores) > 3]

    # IQR method (robust, non-parametric)
    Q1 = df['value'].quantile(0.25)
    Q3 = df['value'].quantile(0.75)
    IQR = Q3 - Q1
    lower = Q1 - 1.5 * IQR
    upper = Q3 + 1.5 * IQR
    outliers_iqr = df[(df['value'] < lower) | (df['value'] > upper)]

    # Isolation Forest (multivariate, unsupervised)
    from sklearn.ensemble import IsolationForest
    iso = IsolationForest(contamination=0.05, random_state=42)
    outlier_labels = iso.fit_predict(X)  # -1 = outlier
    ```

    **Handling strategies:**

    - **Remove**: only if you're confident it's a data error and not a real event.
    - **Cap/winsorise**: clip values at the 1st and 99th percentiles. Keeps the row but limits influence.
    - **Transform**: log or square root transformation naturally compresses extreme values.
    - **Separate model**: flag outliers and model them separately if they represent a distinct population.
    - **Use robust models**: tree-based models are naturally resistant to outliers. MAE loss is more robust than MSE.

    The right choice depends on whether the outlier is noise or signal.

---

### Q16: What is the difference between filter, wrapper, and embedded feature selection methods?

??? "Show answer"
    **Filter methods** evaluate features independently of the model using statistical measures. Fast and scalable, but don't account for feature interactions.

    - Variance threshold: remove near-zero variance features.
    - Correlation: remove features highly correlated with others.
    - Mutual information, chi-squared test, ANOVA F-score.

    ```python
    from sklearn.feature_selection import SelectKBest, mutual_info_classif, VarianceThreshold

    # Remove low-variance features
    vt = VarianceThreshold(threshold=0.01)
    X_filtered = vt.fit_transform(X)

    # Select top 20 by mutual information
    selector = SelectKBest(mutual_info_classif, k=20)
    X_best = selector.fit_transform(X, y)
    ```

    **Wrapper methods** evaluate subsets of features by training a model on each subset and measuring validation performance. Accounts for feature interactions but is computationally expensive.

    - Forward selection, backward elimination, recursive feature elimination (RFE).

    ```python
    from sklearn.feature_selection import RFE
    from sklearn.linear_model import LogisticRegression

    rfe = RFE(estimator=LogisticRegression(), n_features_to_select=15, step=1)
    X_rfe = rfe.fit_transform(X, y)
    ```

    **Embedded methods** perform feature selection as part of model training. The model inherently assigns zero or near-zero weights to irrelevant features.

    - Lasso regression (L1 penalty zeroes out coefficients).
    - Random Forest feature importance.
    - XGBoost/LightGBM gain-based importance.

    ```python
    from sklearn.linear_model import LassoCV

    lasso = LassoCV(cv=5).fit(X_scaled, y)
    selected = X.columns[lasso.coef_ != 0]
    ```

    **In practice:** use filter methods for initial pruning of obviously useless features, embedded methods for the main selection, and wrapper methods only when you need the maximum performance and have the compute budget.

---

### Q17: What is a feature store?

??? "Show answer"
    A feature store is a centralised platform for storing, managing, sharing, and serving machine learning features across teams and models.

    **The problem it solves:** in large organisations, the same feature (e.g., "user's 30-day average spend") gets independently computed by multiple teams, creating inconsistencies, duplicated work, and training-serving skew. A feature store ensures that the feature computed during training is identical to the feature served at inference.

    **Core capabilities:**

    - **Feature registry**: a catalogue of all features, their definitions, ownership, and lineage.
    - **Offline store**: historical feature values for model training (typically a columnar store like S3 + Parquet, or a data warehouse).
    - **Online store**: low-latency feature retrieval for real-time inference (typically Redis, DynamoDB, or Cassandra).
    - **Point-in-time correct joins**: when training, retrieve the exact feature value that would have been available at the time of each training label — avoids future leakage in historical data.

    ```python
    # Example using Feast (open-source feature store)
    from feast import FeatureStore

    store = FeatureStore(repo_path=".")

    # Offline retrieval for training
    training_df = store.get_historical_features(
        entity_df=entity_df,
        features=["user_stats:avg_spend_30d", "user_stats:n_orders_7d"]
    ).to_df()

    # Online retrieval for inference
    feature_vector = store.get_online_features(
        features=["user_stats:avg_spend_30d"],
        entity_rows=[{"user_id": "user_123"}]
    ).to_dict()
    ```

    Feature stores matter when: multiple models share features, real-time serving is required, maintaining consistency between training and serving is critical, or you need auditing and lineage tracking for regulatory compliance.

---
