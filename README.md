# What Makes Power Outages Last So Long?

**Name**: Calvin Chen

---

## Introduction

This project analyzes a dataset of major power outages in the United States from January 2000 to July 2016. The dataset contains 1534 rows, where each row represents a single major power outage event. 

The central question I am investigating is: **what characteristics are associated with the duration of power outages?** This question matters because outage duration directly determines the economic and social impact of a blackout. Understanding what drives longer outages could help utility companies better allocate restoration resources and help policymakers prioritize grid improvements.

The columns most relevant to my analysis are:

| Column | Description |
|:-------|:------------|
| `OUTAGE.DURATION` | Duration of the outage in minutes |
| `CAUSE.CATEGORY` | General category of the cause of the outage |
| `CUSTOMERS.AFFECTED` | Number of customers affected by the outage |
| `CLIMATE.REGION` | U.S. climate region where the outage occurred |
| `U.S._STATE` | State where the outage occurred |
| `SEASON` | Season the outage started (derived from start month) |
| `POPDEN_URBAN` | Population density of urban areas in the state |
| `AREAPCT_URBAN` | Percentage of the state's area that is urban |

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

I performed the following cleaning steps on the raw dataset:

1. Combined `OUTAGE.START.DATE` and `OUTAGE.START.TIME` into a single `OUTAGE.START` timestamp column, and did the same for `OUTAGE.RESTORATION.DATE` and `OUTAGE.RESTORATION.TIME`. This made it easier to work with the temporal data and allowed me to derive a `SEASON` column.
2. Replaced `OUTAGE.DURATION` values of 0 with `NaN` since a real outage cannot last 0 minutes — these likely represent missing data rather than instantaneous events.
3. Created a `SEASON` column by mapping the start month to Winter, Spring, Summer, or Fall. This was useful for EDA and later for modeling.
4. Dropped the original split date and time columns after combining them.

Here is the head of the cleaned DataFrame:

|    |   YEAR |   MONTH | U.S._STATE   | CAUSE.CATEGORY     |   OUTAGE.DURATION |   CUSTOMERS.AFFECTED | CLIMATE.REGION     | SEASON   |
|---:|-------:|--------:|:-------------|:-------------------|------------------:|---------------------:|:-------------------|:---------|
|  0 |   2011 |       7 | Minnesota    | severe weather     |              3060 |                70000 | East North Central | Summer   |
|  1 |   2014 |       5 | Minnesota    | intentional attack |                 1 |                  nan | East North Central | Spring   |
|  2 |   2010 |      10 | Minnesota    | severe weather     |              3000 |                70000 | East North Central | Fall     |
|  3 |   2012 |       6 | Minnesota    | severe weather     |              2550 |                68200 | East North Central | Summer   |
|  4 |   2015 |       7 | Minnesota    | severe weather     |              1740 |               250000 | East North Central | Summer   |

### Univariate Analysis

<iframe
  src="assets/outage-duration-dist.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

The distribution of outage duration is heavily right-skewed, with most outages lasting under a few thousand minutes but a small number of extreme outliers stretching to over 100,000 minutes. I used a log y-scale to make the distribution more readable.

<iframe
  src="assets/outage-cause-counts.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

Severe weather is by far the most common cause of major outages, accounting for nearly half of all events in the dataset. Intentional attacks are the second most common cause.

### Bivariate Analysis

<iframe
  src="assets/outage-duration-by-cause.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

Severe weather and fuel supply emergencies tend to produce the longest outages by a significant margin. Intentional attacks, while frequent, tend to be resolved relatively quickly compared to weather-driven events.

### Interesting Aggregates

The table below shows the mean outage duration (in minutes) broken down by cause category and season.

| CAUSE.CATEGORY                |   Winter |   Spring |   Summer |   Fall |
|:------------------------------|---------:|---------:|---------:|-------:|
| equipment failure             |    712.8 |   4959.8 |    292.8 |  389.5 |
| fuel supply emergency         |  18935.9 |  16503.3 |   7626.2 |  793.7 |
| intentional attack            |    490.8 |    782.3 |    331.8 |  334.8 |
| islanding                     |    385.0 |     88.4 |    231.9 |   69.0 |
| public appeal                 |   2592.6 |   2152.3 |   1187.6 |  244.0 |
| severe weather                |   3869.3 |   3287.7 |   3214.8 | 5646.1 |
| system operability disruption |    592.3 |    529.9 |   1152.1 |  371.5 |

Fuel supply emergencies are dramatically longer in winter and spring, which makes intuitive sense since heating demand peaks in colder months and stresses fuel supply chains. Severe weather outages are longest in fall, likely driven by hurricane season along the East Coast.

---

## Assessment of Missingness

### MNAR Analysis

I believe `DEMAND.LOSS.MW` is likely **MNAR**. Measuring demand loss in megawatts requires real-time grid instrumentation, and smaller utilities or outages in rural areas may simply not have that equipment. This means the missingness is tied to the value itself: the outages most likely to have low demand loss (small, rural events) are also the ones least likely to have the measurement recorded at all. To convert this to MAR, I would want data on whether each affected utility has advanced metering infrastructure, since that column would explain the missingness independently of the demand loss value.

### Missingness Dependency

I analyzed the missingness of `CUSTOMERS.AFFECTED`, which is missing for about 29% of rows. I tested whether this missingness depends on `CAUSE.CATEGORY` and `YEAR`.

**Missingness does depend on `CAUSE.CATEGORY`** (p-value = 0.0): intentional attack outages are much more likely to have missing customer counts than severe weather outages, which makes sense since targeted attacks often affect very few or zero customers and may not be tracked the same way.

**Missingness also depends on `YEAR`** (p-value = 0.0): earlier years in the dataset are more likely to have missing customer data, suggesting that reporting standards improved over time.

<iframe
  src="assets/missingness-tvd.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

The observed TVD of 0.557 is far outside the permutation distribution, confirming that the missingness of `CUSTOMERS.AFFECTED` is strongly dependent on `CAUSE.CATEGORY`.

---

## Hypothesis Testing

**Question**: Do severe weather outages last longer on average than intentional attack outages?

- **Null hypothesis**: Severe weather and intentional attack outages have the same mean duration, and any observed difference is due to random chance.
- **Alternative hypothesis**: Severe weather outages have a longer mean duration than intentional attack outages.
- **Test statistic**: Difference in group means (severe weather minus intentional attack). I used a one-sided test since I had a directional hypothesis based on domain knowledge.
- **Significance level**: 0.05
- **Result**: The observed difference was 3454 minutes, and the p-value was 0.0.

Since the p-value is well below 0.05, I reject the null hypothesis. The data is consistent with the idea that severe weather outages last significantly longer than intentional attack outages on average. This makes sense since weather events tend to cause widespread infrastructure damage that takes days to repair, while targeted attacks are more localized and faster to resolve.

<iframe
  src="assets/hypothesis-test.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

---

## Framing a Prediction Problem

The prediction problem I am focusing on is predicting `OUTAGE.DURATION` (in minutes), which represents how long a power outage lasts from start to restoration. This is a **regression** problem since the response variable is continuous.

I chose outage duration as my response variable because it is one of the most practically important outcomes of a power outage. Longer outages cause more economic damage and affect more people, so understanding what drives duration could help utilities better allocate restoration resources.

To avoid data leakage, I only use features that would be known at the time the outage begins, before restoration occurs. This means I cannot use anything recorded after the fact like `OUTAGE.RESTORATION` or `DEMAND.LOSS.MW`. The features I use describe the cause, location, climate, and regional context of the outage.

I use **RMSE (Root Mean Squared Error)** as my evaluation metric because it is interpretable in the same units as the response variable (minutes) and penalizes large prediction errors more heavily, which matters here since very long outages tend to be the most impactful.

---

## Baseline Model

My baseline model predicts `OUTAGE.DURATION` using a `RandomForestRegressor` inside a single sklearn `Pipeline`. I used two features:

- `CAUSE.CATEGORY` (nominal): the cause of the outage, encoded with `OneHotEncoder` since it has no natural ordering
- `CUSTOMERS.AFFECTED` (quantitative): the number of customers affected, left as-is after imputing missing values with the median

I chose these two features because from my EDA, cause category showed a very clear relationship with duration, and customers affected gives a sense of the scale of the outage.

The baseline model achieved a **train RMSE of 4347.21 minutes** and a **test RMSE of 7087.31 minutes**. The large gap between train and test RMSE indicates the model is overfitting, and the high test RMSE overall is partly driven by extreme outliers in outage duration. I would not consider this a good model yet since it only uses two features without any thoughtful transformations and the hyperparameters have not been tuned at all.

---

## Final Model

For my final model I added the following new features on top of the two from the baseline:

- **Log transform of `CUSTOMERS.AFFECTED`**: outage duration and customer counts are both heavily right-skewed, so a log transformation helps linearize the relationship. The intuition is that doubling the number of affected customers does not double the restoration time, the relationship is more logarithmic in nature.
- **`CLIMATE.REGION`** (nominal): different regions have very different grid infrastructure and utility response capacity. A Northeast outage and a Southwest outage of the same cause are likely to have very different restoration times, so this adds meaningful regional signal.
- **`SEASON`** (nominal): from my EDA, fall severe weather outages lasted around 5646 minutes on average compared to around 3215 in summer. Season clearly affects how bad weather-related damage is and how quickly crews can respond.
- **`POPDEN_URBAN`** (quantitative): more densely populated urban areas tend to have more redundant grid infrastructure, meaning outages there can often be resolved faster. I used a `StandardScaler` to normalize the range.

I also capped extreme outliers at the 99th percentile of duration before modeling, since outages like the 108,000 minute event in the dataset are essentially impossible to predict from the available features and were artificially inflating RMSE.

I used a `RandomForestRegressor` and tuned `max_depth`, `n_estimators`, and `min_samples_split` using `GridSearchCV` with 5-fold cross validation. The best hyperparameters found were `max_depth=10`, `min_samples_split=10`, and `n_estimators=100`.

The final model achieved a **train RMSE of 2304.90 minutes** and a **test RMSE of 2548.13 minutes**, compared to the baseline test RMSE of 7087.31 minutes. This is an improvement of about 4539 minutes, or roughly a 64% reduction in test RMSE. The train/test gap also narrowed significantly, indicating much less overfitting.

---

## Fairness Analysis

I tested whether my final model performs fairly across outages in highly urbanized vs. less urbanized states, split by the median of `AREAPCT_URBAN`.

- **Group X (high urbanization)**: outages where `AREAPCT_URBAN` is above the median
- **Group Y (low urbanization)**: outages where `AREAPCT_URBAN` is at or below the median
- **Evaluation metric**: RMSE
- **Null hypothesis**: the model is fair. Its RMSE for high urbanization and low urbanization outages are roughly the same, and any differences are due to random chance.
- **Alternative hypothesis**: the model is unfair. Its RMSE is higher for low urbanization outages than high urbanization outages.
- **Test statistic**: difference in RMSE (low urbanization RMSE minus high urbanization RMSE)
- **Significance level**: 0.05
- **Result**: observed difference = 628.28 minutes, p-value = 0.196

Since the p-value is greater than 0.05, I fail to reject the null hypothesis. This suggests that my model is fair with respect to urbanization level — the difference in performance between high and low urbanization outages is not statistically significant and could reasonably be due to random chance.

<iframe
  src="assets/fairness-permutation.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>