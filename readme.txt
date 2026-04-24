Python Libraries used:
1. numpy
2. pandas
3. matplotlib.pyplot
4. sklearn.linear_model
5. sklearn.metrics
6. xgboost

Workflow:
1. Data Ingestion: I loaded the 3-dataset provided into the jupyter notebook.

2. Data Cleaning:
a. A scatter plot was plotted to check for duplicate timestamps, then the duplicate timestamp issue was resolved by aggregating the data using the mean for each timestamp. (I did think about using different aggregate functions for different columns, but only mean made sense to me)
b. A bar diagram was plotted to check for irregularity in the frequency of timestamps. Irregular timestamp intervals were standardized by converting the data to a fixed hourly frequency using “asfreq(‘h’)”, followed by forward filling to handle missing values without introducing data leakage.
c. Missing values were handled using a combination of statistical and domain-specific approaches:
•	Features such as Nepal, remarks, and wind were dropped due to a high proportion of missing values (>80%) and low correlation with the target. 
•	The India Adani feature, despite high missingness, was retained due to strong correlation (>60%) and a meaningful contribution (~7.74%) to demand. Missing values were imputed as zero. 
•	Solar data was handled using domain knowledge by assigning zero values during nighttime and filling remaining gaps using past-based methods. 
•	Remaining features were imputed using forward fill and rolling mean techniques, ensuring smooth and realistic value propagation.
d. Extreme spikes in demand were observed and treated as anomalies. These were handled using percentile-based clipping (1st–99th percentile) to remove unrealistic values while preserving the overall trend.
NOTE:  Data was sorted chronologically before applying time-series operations. And Interpolation methods were avoided as they utilize future values and can introduce data leakage in time-series forecasting.


3. Data Integration
Economic and weather datasets were integrated with the primary power dataset to enrich the feature space with external influencing factors.
a.	Economic Data Integration:
Economic indicators including GDP, population, urban population percentage, electricity access, electricity consumption per capita, industry percentage, and services percentage were incorporated. Since economic data is available at a yearly granularity, the year attribute was used as a key to merge it with the hourly power dataset, ensuring temporal alignment at an annual level. 
b.	Weather Data Integration:
Relevant weather features were selected based on their potential impact on electricity demand, including temperature (2m), relative humidity, apparent temperature, cloud cover, and sunshine duration. These features were aligned and merged with the power dataset using the timestamp index, ensuring precise temporal correspondence.

4. Feature Engineering
a.	Extracted time-based features such as hour, day of week, and month to capture temporal patterns. 
b.	Created multiple lag features (lag_1, lag_2, lag_3, lag_6, lag_12, lag_24, lag_48, lag_168) to model short-term, daily, and weekly dependencies in electricity demand. 
c.	Introduced difference features (diff_1, diff_24) to capture rate of change and short-term variations in demand. 
d.	Computed rolling statistical features, including moving averages (3-hour, 12-hour, 24-hour) and rolling standard deviation (24-hour) to capture local trends and demand volatility. 
e.	Defined the target variable as next-hour demand using shift(-1). 
f.	Constructed a thermal index by combining temperature, apparent temperature, and humidity. Z-score normalization was applied prior to aggregation to account for differences in scale.

5. Model Development:
a. Feature selection was performed by excluding the target variable and current demand to prevent data leakage, while retaining all relevant engineered and external features. The dataset was split using a time-based approach, with data before 2023 used for training and data from 2023 onwards used for testing, ensuring realistic forecasting conditions.
b. Linear Regression model was first implemented as a baseline to establish a reference performance. Subsequently, an XGBoost regressor was employed as the primary model due to its ability to capture nonlinear relationships and complex feature interactions.
c. Further improvements were made by tuning the XGBoost model, enhancing its generalization and predictive accuracy
d. Models were trained on the selected features and evaluated on the test set to compare performance.

6. Model Evaluation: Model performance was evaluated using Mean Absolute Percentage Error (MAPE) to measure prediction accuracy relative to actual demand values. The Linear Regression model yielded a MAPE of 7.7%, indicating limited capability in capturing nonlinear relationships and complex temporal dependencies. The baseline XGBoost model significantly improved performance, achieving a MAPE of 3.14%, demonstrating its effectiveness in modelling nonlinear interactions and feature dependencies. Further hyperparameter tuning and refinement led to an improved XGBoost model with a reduced MAPE of 3.09%

