## B1. Problem Formulation

### (a) Problem Formulation

This problem can be framed as a supervised regression problem.

* **Target variable:**
  `items_sold` — number of products sold per store in a given time period

* **Input features:**

  * Store characteristics: store_size, location_type, competition_density
  * Promotion-related: promotion_type
  * Time-based features: month, seasonality
  * Demand signals: footfall, demographics (if available)

Since we are predicting a numerical value (number of items sold), this is a regression task. The idea is to estimate how different promotions perform across stores and pick the one that leads to higher sales.

---

### (b) Why use items_sold instead of revenue

Using `items_sold` is more reliable than revenue because revenue can get affected by pricing and discounts.

For example, selling fewer expensive items might give high revenue but doesn’t really mean the promotion worked well. On the other hand, items sold directly shows customer response.

So, items_sold is a better measure of demand and engagement.

**Key idea:**
The target variable should match the business goal and should not be distorted by the decision itself (like discounts affecting revenue).

---

### (c) Alternative Modelling Strategy

Instead of using one single model for all stores, it’s better to use a segmented approach.

For example:

* Separate models for urban, semi-urban, and rural stores, or
* A single model with interaction between store type and promotion

This makes sense because customer behavior is different across locations. A single model might average everything out and miss these differences.

---

## B2. Data and EDA Strategy

### (a) Data Joining and Structure

We need to combine multiple tables into one dataset:

* Transactions → base table
* Store attributes → join using store_id
* Promotions → join using promotion_type or ID
* Calendar → join using transaction_date

After joining, the data should be aggregated.

* **Final grain:**
  One row per store per time period (ideally monthly)

* **Aggregations:**

  * Total items_sold (target)
  * Footfall (if available)
  * Promotion used
  * Calendar features (weekend/festival)

This ensures the dataset matches how decisions are actually made.

---

### (b) EDA Approach

Before building the model, I would look at:

1. **Sales distribution (histogram / boxplot):**
   To check spread and outliers

2. **Promotion vs sales (bar chart):**
   To see which promotions perform better

3. **Time trends (line plot):**
   To identify seasonality

4. **Correlation heatmap:**
   To understand relationships between variables

5. **Store-level comparison:**
   Compare urban vs rural vs semi-urban

These help in understanding the data and deciding what features to use.

---

### (c) Handling Imbalance

If 80% of data has no promotion, the model may just learn baseline behavior and ignore promotions.

To fix this:

* Ensure promotion cases are properly represented
* Evaluate separately for promo vs non-promo
* Add features to capture promotion impact
* Use sampling if needed

---

## B3. Model Evaluation and Deployment

### (a) Train-Test Split and Metrics

Since the data is time-based, we should use a **time-based split**:

* First ~2–2.5 years → training
* Last ~6–12 months → testing

Random split is not suitable because it mixes past and future data (data leakage).

**Metrics:**

* **RMSE:** penalizes large errors
* **MAE:** easy to interpret (average error in items)

For example, MAE = 20 means predictions are off by ~20 items on average.

---

### (b) Explaining Model Recommendations

If the model suggests different promotions for the same store in different months, it’s because the underlying conditions change.

For example:

* December → high demand → loyalty rewards may work better
* March → lower demand → discounts may be more effective

In simple terms:

> The model looks at timing + store behavior, not just the store itself.

---

### (c) Deployment

Once the model is trained:

1. Save the model (using joblib/pickle)
2. Each month, collect new data
3. Apply same preprocessing
4. Generate predictions for all promotions
5. Choose the best one per store

**Monitoring:**

* Track MAE/RMSE over time
* Check if predictions start getting worse
* Retrain when needed

---

### Final Note

Overall, the goal is to use data to make better promotion decisions at a store level, while keeping the model simple, interpretable, and practical to deploy.
