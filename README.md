**By: Romy Ramrakhyani**

## Introduction
Power outages are a complex phenomenon influenced not just by external events like severe weather, but also by the underlying infrastructure and economic demands of the region. This dataset, from Purdue University, contains records of major power outages in the continental U.S. from January 2000 to July 2016. Alongside the temporal and geographic data of these outages, the dataset provides state level information on electricity consumption patterns, including residential, commercial, and industrial sales.  

My project will be framed around the question: **"Is there a relationship between a state's electricity consumption patterns (such as the ratio of commercial to residential sales) and the severity of the power outages they experience?"** If states with heavy industrial or commercial loads experience longer or more severe outages, it may indicate that certain types of power grid infrastructure struggle to recover under specific economic demands. Energy companies could use this information to better allocate resources and fortify the grid in highly commercial or industrial zones.

The columns relevant to my analysis are:
* **`OUTAGE.DURATION`**: The duration of the outage in minutes. (This is my primary severity metric).
* **`COM.PROP`**: The proportion of the state's total electricity sales that go to commercial customers.
* **`RES.PROP`**: The proportion of the state's total electricity sales that go to residential customers.
* **`IND.PROP`**: The proportion of the state's total electricity sales that go to industrial customers.
* **`CLIMATE.CATEGORY`**: The climate classification of the state (e.g., 'warm', 'cold', 'normal').
* **`DEMAND.LOSS.MW`**: The peak demand lost during the outage in Megawatts.

## Data Cleaning and Exploratory Data Analysis
To prepare the data for analysis, I performed the following cleaning steps:
1. Combined the separate `DATE` and `TIME` columns for both outage start and restoration into single, easy-to-use `datetime` objects.
2. Cleaned severity columns (like `CUSTOMERS.AFFECTED` and `OUTAGE.DURATION`) by converting 0s to missing values (`NaN`), as an outage cannot reasonably affect 0 customers or last 0 minutes.
3. Created proportion columns (`RES.PROP`, `COM.PROP`, `IND.PROP`) to measure the economic makeup of the state's electricity usage.
4. Extracted the exact hour the outage started and calculated the outage duration in minutes.

Here are the first few rows of the cleaned dataset:

| U.S._STATE   |   RES.PROP |   COM.PROP |   IND.PROP |   OUTAGE.START.HOUR |   CALCULATED.DURATION |
|:-------------|-----------:|-----------:|-----------:|--------------------:|----------------------:|
| Minnesota    |   0.355491 |   0.32225  |   0.322024 |                  17 |                  3060 |
| Minnesota    |   0.300325 |   0.342104 |   0.357276 |                  18 |                     1 |
| Minnesota    |   0.280977 |   0.34501  |   0.37366  |                  20 |                  3000 |
| Minnesota    |   0.319941 |   0.335433 |   0.344393 |                   4 |                  2550 |
| Minnesota    |   0.339826 |   0.362059 |   0.297795 |                   2 |                  1740 |

### Univariate Analysis
First, I looked at the individual distributions of our primary features: **Commercial Electrical Sales Proportion (`COM.PROP`)** and **Outage Duration (`OUTAGE.DURATION`)**.  
<iframe src="assets/fig1_com_prop_dist.html" width="800" height="600" frameborder="0"></iframe>

<iframe src="assets/fig2_outage_dur_dist.html" width="800" height="600" frameborder="0"></iframe>

### Bivariate Analysis
Next, I explored the relationship between the commercial sales proportion and the duration of the outages using a scatter plot and a box plot.
<iframe src="assets/fig3_bivariate_scatter.html" width="800" height="600" frameborder="0"></iframe>

<iframe src="assets/fig4_bivariate_box.html" width="800" height="600" frameborder="0"></iframe>

### Aggregates
Grouping the data by High vs. Low commercial footprint reveals interesting differences in the average and median outage durations. Surprisingly, the data shows that states with a **High Commercial** footprint actually experience noticeably *shorter* median outage durations (622 minutes) compared to **Low Commercial** states (1098.5 minutes).

| COM_CATEGORY    | Count | Median Duration | Mean Duration | Max Duration | Mean COM.PROP |
|:----------------|:------|:----------------|:--------------|:-------------|:--------------|
| High Commercial | 694   | 622.0           | 2721.74       | 78377.0      | 0.437645      |
| Low Commercial  | 704   | 1098.5          | 2821.30       | 108653.0     | 0.308217      |

I broke this down further by the time of day the outage started:

| COM_CATEGORY    | Night (Midnight-5AM) | Morning (6AM-11AM) | Afternoon (Noon-5PM) | Evening (6PM-Midnight) |
|:----------------|:---------------------|:-------------------|:---------------------|:-----------------------|
| High Commercial | 1185.0               | 420.0              | 396.0                | 1140.0                 |
| Low Commercial  | 1919.0               | 517.5              | 510.0                | 1430.0                 |


## Assessment of Missingness

### NMAR Analysis
I believe the `DEMAND.LOSS.MW` (Demand Loss in Megawatts) column is potentially **NMAR (Not Missing At Random)**. The data generating process for this column likely relies on the utility company's ability to accurately monitor grid loads during an ongoing crisis. If an outage is incredibly severe or catastrophic, the physical monitoring equipment itself may lose power, or the utility workers may be too overwhelmed with emergency response to accurately record the demand drop. Therefore, the missingness depends on the actual (unrecorded) severity of the demand loss itself. To make this data MAR (Missing At Random), we would need to obtain additional data, such as a "Severe Weather Category" or the "Number of Active Utility Workers Deployed", which could explain the lack of monitoring capacity.

### Missingness Dependency
I evaluated whether the missingness of the `DEMAND.LOSS.MW` column was dependent on other columns in the dataset using permutation tests. For each test, I used the **Difference in Means** as the test statistic to determine if the distribution of a secondary variable differed significantly when `DEMAND.LOSS.MW` was present versus when it was missing.

**Test 1: Commercial Proportion (COM.PROP)**
I ran a permutation test to see if the missingness of Demand Loss depends on the commercial sales proportion of the state. With a p-value of 0.0190 (which is less than my significance level of 0.05), we reject the null hypothesis. The missingness of Demand Loss is dependent on the commercial proportion (MAR).

<iframe src="assets/fig_missingness_com.html" width="800" height="600" frameborder="0"></iframe>

**Test 2: Residential Proportion (RES.PROP)**
I ran a second test against the residential proportion. With a p-value of 0.8140 (which is greater than my significance level of 0.05), we fail to reject the null hypothesis. The missingness of Demand Loss does not appear to depend on the residential proportion.

<iframe src="assets/fig_missingness_res.html" width="800" height="600" frameborder="0"></iframe>


## Hypothesis Testing
To formally answer my research question, I ran a permutation test to determine if there is a statistically significant correlation between a state's commercial electricity proportion (`COM.PROP`) and the duration of its outages (`OUTAGE.DURATION`).

* **Null Hypothesis:** There is no correlation between commercial proportion and outage duration. Any observed correlation is due to random chance.
* **Alternative Hypothesis:** There is a correlation between commercial proportion and outage duration.
* **Test Statistic:** Absolute Pearson Correlation Coefficient (|r|). I chose this because it captures any linear association (positive or negative) between two quantitative variables.
* **Significance Level:** 0.05

<iframe src="assets/fig_hypothesis_test.html" width="800" height="600" frameborder="0"></iframe>

**Conclusion:** The observed correlation was 0.0072, yielding a p-value of 0.7830. Since the p-value is greater than our 0.05 significance level, we fail to reject the null hypothesis. The data suggests there is no statistically significant linear correlation between a state's commercial electricity proportion and the duration of its power outages.


## Framing a Prediction Problem
For my prediction problem, I am building a machine learning model to estimate the duration of a major power outage in minutes. This is a regression task because the target variable I am trying to predict, `OUTAGE.DURATION`, is a continuous numerical value. I chose this specific response variable because understanding exactly how long the power grid will be down is critical for emergency management, hospital resource allocation, and utility repair scheduling.

To ensure this model is practical for real-world application, it will only use features that are strictly known at the exact moment an outage begins. These predictive features will include the region's climate category, the economic makeup of the area (such as the proportion of commercial, residential, and industrial electricity sales), and the specific time of day the outage started. Crucially, any information that would only be known after the fact—such as the actual restoration time—must be excluded from the model to prevent data leakage.

Finally, I will use Root Mean Squared Error (RMSE) to evaluate the model's performance. Because outage durations can vary wildly from a few minutes to several days, RMSE is an ideal choice since it heavily penalizes large prediction errors.


## Baseline Model
My baseline model is a Linear Regression model trained to predict the outage duration. 
* **Features Used:** * `CLIMATE.CATEGORY`: A **nominal** categorical feature, which I One-Hot Encoded.
  * `COM.PROP`: A **quantitative** continuous feature, which I left as-is (passed through).
* **Performance:** The model achieved an RMSE of 4112.15 minutes and an R² of -0.0077. 

I do not believe this is a "good" model. Because power outages are highly volatile and heavily influenced by unpredictable severe weather events, these basic regional metrics alone struggle to accurately capture the massive variance in recovery times, resulting in a negative R² score.

## Final Model
For the final model, I upgraded to a `RandomForestRegressor` and engineered two new sets of features to improve upon the baseline:
1. `OUTAGE.START.HOUR`: Transformed using a `KBinsDiscretizer` to group the start times into categorical bins (Night, Morning, Afternoon, Evening). **Why:** Electricity usage is non-linear; it peaks during the day and drops at night. Binning allows the model to treat the time of day categorically rather than as a strict numerical scale.
2. `RES.PROP` and `IND.PROP`: Added alongside the commercial proportion and transformed using a `QuantileTransformer`. **Why:** Outage durations and commercial/industrial proportions have heavy outliers. The Quantile Transformer smooths out these skewed outlier regions and maps the data to a uniform distribution, preventing extreme states from dragging the model's weights.

I performed a Grid Search using 5-fold cross-validation to tune the hyperparameters, finding the optimal settings to be `max_depth: 5` and `min_samples_split: 2`. 

**Performance:** The final model achieved an RMSE of 4142.32 minutes and an R² of -0.0225. While the complex feature engineering was successfully implemented, the model's performance did not see a mathematical improvement over the baseline. This reinforces the conclusion that power outage durations are dictated by unpredictable, localized physical factors (like a tree falling on a wire) rather than high-level regional economic data. 


## Fairness Analysis
Finally, I evaluated if my model's performance is fair with respect to the economic makeup of the affected regions. Specifically, I tested if the model predicts outage durations more accurately for areas with high commercial electricity usage versus low commercial usage.

* **Group X:** "High Commercial" proportion (`COM.PROP` > median)
* **Group Y:** "Low Commercial" proportion (`COM.PROP` <= median)
* **Evaluation Metric:** Root Mean Squared Error (RMSE)

**Null Hypothesis:** Our model is fair. Its RMSE for High Commercial areas and Low Commercial areas are roughly the same, and any observed differences are due to random chance.

<iframe src="assets/fig_fairness_test.html" width="800" height="600" frameborder="0"></iframe>

**Conclusion:** I ran a permutation test with 500 simulations. The resulting p-value is 0.3580, which is well above our significance level of 0.05. Therefore, I **fail to reject the null hypothesis**. The model is fair; the difference in RMSE between high-commercial (3752.73 mins) and low-commercial (4498.30 mins) areas is not statistically significant and is likely just due to random chance.
