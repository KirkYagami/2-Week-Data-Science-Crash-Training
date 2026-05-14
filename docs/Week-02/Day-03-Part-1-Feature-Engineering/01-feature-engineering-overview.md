# 🛠️ 01 — Feature Engineering Overview

Feature engineering is the process of turning raw data into model-ready input.

Good features often matter more than fancy models.

---

## Examples

| Raw Data | Engineered Feature |
|---|---|
| order_date | month, day_of_week, is_weekend |
| signup_date | customer_tenure_days |
| transaction_amount | log_amount |
| city | one-hot encoded city |
| review_text | TF-IDF vectors |

---

## Feature Engineering Workflow

1. Understand the problem.
2. Identify feature types.
3. Clean invalid values.
4. Create useful transformations.
5. Encode categorical variables.
6. Scale when needed.
7. Build pipeline.
8. Validate no leakage.

---

## Guiding Rule

Only use information available at prediction time.

If a feature would not exist when the model is used, it is leakage.

---

## Next

➡️ [[02-numeric-features]]
