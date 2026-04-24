### B1-PROBLEM FORMULATION 

### (a) Problem Formulation:

This problem can be formulated as a supervised machine learning regression problem.

The target variable is `items_sold`, which represents the number of products sold at a store during a given time period.

The candidate input features include:
- Store characteristics: store_size, location_type, competition_density
- Promotion-related variables: promotion_type
- Temporal features: month, seasonality indicators
- Customer and demand signals: footfall, demographics (if available)

This is a regression problem because the goal is to predict a continuous numerical value (number of items sold). The model will learn patterns between promotions, store characteristics, and sales volume to recommend the most effective promotion for each store.


### (b) Target Variable Justification:

Using `items_sold` (sales volume) is more reliable than total revenue because revenue can be influenced by factors such as pricing strategies, discounts, and product mix. For example, a high-value item sold in small quantities may generate high revenue but does not necessarily indicate strong customer engagement or promotion effectiveness.

In contrast, items sold directly reflects customer response to promotions and provides a clearer measure of demand and engagement.

This illustrates a broader principle in machine learning: the target variable should closely align with the business objective. In this case, the objective is to maximise customer purchases and promotion effectiveness, which is better captured by sales volume rather than revenue.


### (c) Alternative Modelling Strategy:

Instead of using a single global model across all 50 stores, a better approach would be to use a segmented or hierarchical modelling strategy.

Stores can be grouped based on characteristics such as location type (urban, semi-urban, rural), store size, or customer demographics. Separate models can then be trained for each segment, or store-specific features can be incorporated into a more advanced model.

This approach accounts for the fact that customer behaviour and promotion effectiveness vary across different store types. For example, urban customers may respond better to loyalty programs, while rural customers may prefer discounts.

By capturing these differences, the model can provide more accurate and personalized promotion recommendations for each store, leading to better business outcomes.


### B2-DATA AND EDA STRATEGY 

### (a) Data Joining and Dataset Structure:

Since the data is coming from multiple tables, the first step would be to combine them into a single modelling dataset.

- The **transactions table** would act as the base since it contains actual sales data.
- The **store attributes table** can be joined using `store_id`.
- The **promotion details table** can be joined using `promotion_type` or promotion ID.
- The **calendar table** can be joined using `transaction_date` to bring in features like weekend and festival flags.

After joining, the grain of the dataset should be:
**one row per store per time period (e.g., per day or per month)**

Before modelling, aggregation will be required. For example:
- Total `items_sold` per store per time period (target variable)
- Average or total `footfall`
- Promotion applied during that period
- Store-level features remain constant



### (b) Exploratory Data Analysis (EDA):

Before building the model, I would perform several analyses to understand patterns in the data:

1. **Sales Distribution Plot (Histogram / Boxplot):**  
   To understand how `items_sold` is distributed and check for outliers.  
   This helps decide if transformations (like log scaling) are needed.

2. **Promotion vs Sales (Bar Chart):**  
   Compare average items sold across different promotion types.  
    Helps identify which promotions are generally more effective.

3. **Time-based Trend (Line Plot):**  
   Plot sales over time to identify seasonality or trends.  
   This helps in creating time-based features like month or festival effects.

4. **Correlation Heatmap:**  
   To see relationships between numerical variables like competition_density, footfall, and sales.  
   Helps in feature selection and identifying multicollinearity.

5. **Store Segmentation Analysis:**  
   Compare performance across urban, semi-urban, and rural stores.  
   This helps decide whether segmentation or separate models are needed.



### (c) Handling Data Imbalance:

If 80% of transactions occur without any promotion, the dataset is highly imbalanced. This can cause the model to become biased towards predicting outcomes for non-promotional cases and may not learn the true impact of promotions effectively.

To address this, a few steps can be taken:
- Ensure proper representation of promotional cases using techniques like stratified sampling.
- Create additional features that capture promotion impact more clearly.
- Evaluate model performance separately for promotional and non-promotional cases.
- If needed, use resampling techniques to balance the dataset.


### B3-MODEL EVALUATION and DEPLOYMENT

### (a) Train-Test Split and Evaluation:

Since the data is time-based (monthly data over three years), I would use a time-based split instead of a random split. For example, the first 2–2.5 years of data can be used for training, and the most recent 6–12 months can be used as the test set.

A random split is not appropriate here because it would mix past and future data, leading to data leakage. In a real-world scenario, we always predict future outcomes using past data, so the model should be trained and tested in the same way.

For evaluation, I would use:
- **RMSE (Root Mean Squared Error):** This penalizes larger errors more heavily, which is useful because large forecasting errors can significantly impact business decisions.
- **MAE (Mean Absolute Error):** This gives an average of prediction errors in simple terms and is easier to interpret from a business perspective.

In this context, lower RMSE and MAE indicate better performance. For example, if MAE is 20, it means the model is off by around 20 items on average, which helps the business understand prediction accuracy in practical terms.


### (b) Explaining Model Recommendations:

If the model recommends different promotions for the same store in different months, it is likely due to changes in input features such as seasonality, customer behavior, or external factors.

Using feature importance, I would analyze which variables are driving the model’s predictions. For example, in December, factors like festival season, higher footfall, or holiday shopping trends might make Loyalty Points Bonus more effective. In contrast, in March, lower demand or different customer behavior might make Flat Discount more impactful.

To communicate this to the marketing team, I would explain it in simple terms:
“The model is not just looking at the store, but also at timing and customer behavior. Different months have different patterns, so the best promotion changes accordingly.”

I could also support this with visuals like feature importance charts or example comparisons to make the explanation more intuitive for non-technical stakeholders.


### (c) Deployment Process:

Once the model is trained, it needs to be deployed so that it can generate recommendations every month.

First, the trained model can be saved using tools like joblib or pickle. This allows us to reuse the model without retraining it every time.

Each month, new data (such as store attributes, current promotions, and calendar features) will be collected and preprocessed in the same way as the training data. This data is then passed into the saved model to generate predictions and recommend the best promotion for each store.

For monitoring, I would track model performance over time using metrics like MAE or RMSE. If these metrics start increasing significantly, it may indicate that the model is no longer performing well due to changes in customer behavior or market conditions.

In such cases, retraining the model with more recent data would be necessary to maintain accuracy and relevance.
