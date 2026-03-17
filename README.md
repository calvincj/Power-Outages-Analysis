Here's the full content ready to copy-paste:

```markdown
# What Makes Power Outages Last So Long?

**Name**: Calvin Chen

---

## Introduction

My project looks at major power outages in the United States from January 2000 to July 2016. The dataset has 1534 rows, where each row is one major outage event.

The central question that I'm investigating is: what characteristics are associated with the duration of power outages? 

This matters because outage duration directly determines the economic and social impact of a blackout. I believe that understanding what drives longer outages could help utility companies better allocate restoration resources and help policymakers prioritize grid improvements.

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

I performed the following steps on the raw dataset to clean it:

1. combined `OUTAGE.START.DATE` and `OUTAGE.START.TIME` into a single `OUTAGE.START` timestamp column, and did the same for `OUTAGE.RESTORATION.DATE` and `OUTAGE.RESTORATION.TIME`. This made it easier to work with the time data and also let me create a `SEASON` column.
2. replaced `OUTAGE.DURATION` values of 0 with `NaN` since a real outage can't last 0 minutes. These are probably missing data rather than actual instant events.
3. created a `SEASON` column by mapping the start month to Winter, Spring, Summer, or Fall.
4. dropped the original split date and time columns after combining them.

Here is the head of the cleaned dataset:

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

The distribution of outage duration is heavily right-skewed. Most outages lasted around a few thousand minutes, but a few of them lasted over 100 thousand minutes. I used a log y-scale to make the distribution easier to read.

<iframe
  src="assets/outage-cause-counts.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

Severe weather is the most common cause of major outages by far, as it made up nearly half of all events in the dataset. Intentional attacks is the second most common cause.

### Bivariate Analysis

<iframe
  src="assets/outage-duration-by-cause.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

Severe weather and fuel supply emergencies tended to produce the longest outages by a pretty wide margin. Intentional attacks were frequent, but they tend to get resolved much faster than weather-driven events.

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

Fuel supply emergencies are much longer in winter and spring, which makes sense since heating demand is highest in colder months and puts more stress on fuel supply chains. Severe weather outages are worst in fall, which could be because of the hurricane season in the East Coast.

---

## Assessment of Missingness

### MNAR Analysis

I believe `DEMAND.LOSS.MW` is likely MNAR. I found that measuring demand loss in megawatts requires specialized grid equipment, and smaller or more rural places may not have those equipment. This means that the missingness is actually tied to the value itself. The outages most likely to have low demand loss are also the ones where it probably won't get measured. 

To turn this into MAR, you would need data on whether each utility has that kind of metering infrastructure, since that would explain the missingness on its own without needing to look at the actual demand loss values.

### Missingness Dependency

I looked at the missingness of `CUSTOMERS.AFFECTED`, which is missing for about 29% of rows. I tested whether this missingness is related to `CAUSE.CATEGORY` and `YEAR`.

The missingness does depend on `CAUSE.CATEGORY` (p-value = 0.0): intentional attack outages are much more likely to have missing customer counts than severe weather outages. This makes sense because targeted attacks usually affect very few customers and may not be tracked the same way.

The missingness also depends on `YEAR` (p-value = 0.0): earlier years are more likely to be missing customer data, which suggest that reporting became more consistent over time.

<iframe
  src="assets/missingness-tvd.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

The observed TVD of 0.557 falls way outside the permutation distribution, which confirms that the missingness of `CUSTOMERS.AFFECTED` is strongly tied to `CAUSE.CATEGORY`.

---

## Hypothesis Testing

Question: Do severe weather outages last longer on average than intentional attack outages?

- Null hypothesis: Severe weather and intentional attack outages have the same mean duration, and any difference observed is just due to random chance.
- Alternative hypothesis: Severe weather outages have a longer mean duration than intentional attack outages.
- Test statistic: Difference in group means (severe weather minus intentional attack). I used a one-sided test because I had a directional hypothesis based on what I already knew about the data.
- Significance level: 0.05
- Result: The observed difference was 3,454 minutes, and the p-value was 0.0.

Since the p-value is much less than 0.05, I reject the null hypothesis. The data supports the idea that severe weather outages last significantly longer than intentional attack outages on average. Weather events tend to cause widespread infrastructure damage that takes days to repair, while targeted attacks are more localized and faster to fix.

<iframe
  src="assets/hypothesis-test.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

---

## Framing a Prediction Problem

The prediction problem I'm focusing on is predicting `OUTAGE.DURATION` (in minutes). This is a **regression** problem since the response variable is continuous.

I chose outage duration as my response variable because it is one of the most practically important outcomes of a power outage. Longer outages cause more economic damage and affect more people, so being able to predict duration could help utilities better plan their response.

To avoid data leakage, I only use features that would be known at the time the outage starts, before restoration happens. This means I can't use anything that gets recorded after the fact, like `OUTAGE.RESTORATION` or `DEMAND.LOSS.MW`. The features I use describe the cause, location, climate, and regional context of the outage.

I use RMSE (Root Mean Squared Error) as my evaluation metric because it is in the same units as the response variable (minutes) and penalizes large errors more heavily. That matters here since very long outages tend to be the most impactful.

---

## Baseline Model

My baseline model predicts `OUTAGE.DURATION` using a `RandomForestRegressor` inside a single sklearn `Pipeline`. I used two features:

- `CAUSE.CATEGORY` (nominal): encoded with `OneHotEncoder` since it has no natural ordering
- `CUSTOMERS.AFFECTED` (quantitative): left as-is after filling in missing values with the median

I picked these two features because cause category showed a clear relationship with duration in my EDA, and customers affected gives a rough sense of the scale of the outage.

The baseline model achieved a train RMSE of 4,347 minutes and a **test RMSE of 7,087 minutes**. The big gap between train and test RMSE suggests the model is overfitting. The high test RMSE overall is also partly because of extreme outliers in outage duration. This model isn't good enough yet since it only uses two features without any real transformations and no hyperparameter tuning.

---

## Final Model

For my final model I added the following features on top of the two from the baseline:

- `CAUSE.CATEGORY` (nominal): same as baseline, encoded with `OneHotEncoder`
- `CUSTOMERS.AFFECTED` (quantitative): log-transformed since both outage duration and customer counts are heavily right-skewed. Doubling the number of affected customers doesn't double the restoration time, so a log transformation fits the relationship better.
- `CLIMATE.REGION` (nominal): different regions have very different grid infrastructure and utility response capacity. A Northeast outage and a Southwest outage with the same cause will probably have different restoration times, so this adds useful regional context.
- `SEASON` (nominal): from my EDA, fall severe weather outages lasted around 5,646 minutes on average compared to around 3,215 in summer. Season clearly affects how bad weather-related damage gets and how quickly crews can respond.
- `POPDEN_URBAN` (quantitative): more densely populated urban areas tend to have more backup grid infrastructure, so outages there can often be fixed faster. I applied `StandardScaler` to normalize the range.

I also capped extreme outliers at the 99th percentile of duration before modeling. Outages like the 108,000-minute event in the dataset are basically impossible to predict from the available features and were inflating RMSE in a way that wasn't useful.

I used a `RandomForestRegressor` and tuned `max_depth`, `n_estimators`, and `min_samples_split` using `GridSearchCV` with 5-fold cross validation. The best hyperparameters were `max_depth=10`, `min_samples_split=10`, and `n_estimators=100`.

The final model achieved a train RMSE of 2,305 minutes and a **test RMSE of 2,548 minutes**, compared to the baseline test RMSE of 7,087 minutes. That is roughly a 64% drop in test RMSE. The train/test gap also got much smaller, which means the model is overfitting a lot less.

<iframe
  src="assets/feature-importance.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

The feature importance chart shows which variables the random forest relied on most. Log-transformed customer count is the top predictor, which makes sense since larger outages generally take longer to restore. Cause category features also rank highly, which lines up with the earlier finding that severe weather and fuel supply emergencies drive the longest outages.

<iframe
  src="assets/residual-plot.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

The residual plot compares prediction errors for the baseline and final model on the same test set. Points closer to the horizontal zero line mean more accurate predictions. The final model's errors are more tightly clustered around zero across the range of predicted values, while the baseline model has larger and more spread out errors, especially for longer outages. Both models tend to underpredict very long outages, which is expected since those events are rare and hard to predict.

---

## Fairness Analysis

I tested whether my final model performs fairly across outages in highly urbanized vs. less urbanized states, split by the median of `AREAPCT_URBAN`.

- Group X (high urbanization): outages where `AREAPCT_URBAN` is above the median
- Group Y (low urbanization): outages where `AREAPCT_URBAN` is at or below the median
- Evaluation metric: RMSE
- Null hypothesis: the model is fair. Its RMSE for high urbanization and low urbanization outages are roughly the same, and any difference is just due to random chance.
- Alternative hypothesis: the model is unfair. Its RMSE is higher for low urbanization outages than high urbanization outages.
- Test statistic: difference in RMSE (low urbanization RMSE minus high urbanization RMSE)
- Significance level: 0.05
- Result: observed difference = 628.28 minutes, p-value = 0.196

Since the p-value is above 0.05, I fail to reject the null hypothesis. The difference in performance between the two groups is not statistically significant and could reasonably be due to random chance.

<iframe
  src="assets/fairness-permutation.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

---

## Conclusion

The main takeaway from this project is that outage duration is driven heavily by cause category and geography. Severe weather and fuel supply emergencies produce much longer outages than other causes, and this holds across seasons and regions. The hypothesis test confirmed that severe weather outages last significantly longer than intentional attack outages on average. Season also matters: fall outages tend to be the worst for severe weather, likely because of hurricane season, while winter fuel supply emergencies can stretch close to two weeks.

The final model improved on the baseline by a lot, cutting test RMSE from about 7,087 minutes down to 2,548 minutes. The biggest improvements came from adding `CLIMATE.REGION` and `SEASON` as features, log-transforming customer counts, capping extreme outliers, and tuning hyperparameters. The fairness analysis found no significant difference in model performance between high and low urbanization areas.

However, there is still a lot of room to improve on. The model still struggles with very long outages since those are rare and hard to predict from the available features. Data that isn't in this dataset but would probably help includes the age and size of local grid infrastructure, the number of repair crews available at the time of the outage, and storm severity for weather-related events. Another limitation is that the dataset only goes through 2016, so it may not reflect how grid resilience and utility response have changed since then.
```