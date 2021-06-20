# 1 Summary
The aim of the project is to forecast monthly admissions to Singapore public acute adult hospitals.  The admissions were treated as a hierarchical time series. Admissions were forecasted at each level. Every country has a hierarchical order to its public hospitals. In Singapore, there are 3 levels:

National level
<br>
|-- Cluster level (Clusters are a network of hospitals based on geographical regions. There are 3 health clusters in Singapore- NUHS, NHG, SHS.)
 <br>
|---- Hospital level (There are 8 public acute adult hospitals- NTFGH, NUH, AH, TTSH, KTPH, SGH, SKH, CGH)

Forecasting admissions at hospital levels can help hospital managers plan for better manpower deployment during predicted peak periods. Forecasting at higher levels such as cluster or even national level can help senior management and policy planners develop better strategy to deal with high and lo periods and review other strategies to reduce admission rate. A manageable admission rate helps to ensure clinicians will have sufficient time to review their patients.  

Both classical and machine learning approaches were adopted for forecasting. The best model was ensembled model of retuned Random Forest and retuned Prophet Boost with a 9:1 weighting. This model's accuracy was: 
| Level             | RMSE (testing set) | MAE (testing set) |
|-------------------|------|-----|
| Across all levels | 535  | 412 |
| National          | 949  | 789 |
| Cluster           | 657  | 528 |
| Hospital          | 393 | 321 |

*While smaller rmse are favoured in general, [care needs to be taken when appreciating the rmse for each level as the magnitude of admission differs for each hierarchical level, the superordinate levels have more admissions thus a larger rmse can be expected.](https://otexts.com/fpp3/tourism.html)* 

The datasets, model outputs and key objects are housed on this GitHub. The rest of the README outlines the project, more details are found in [my blog](https://notast.netlify.app/tags/modeltime/)

# 2 Data
The [dataset is monthly admission to Singapore public acute adult hospitals](https://www.tablebuilder.singstat.gov.sg/publicfacing/mainMenu.action). The [dataset starts from Jan 2016 and ends in Feb 2021](https://github.com/notast/hierarchical-forecasting/blob/main/stat_sg.csv). The forecast horizon was 10months, i.e. to forecast till the end of 2021. The training set was from Jan 16 to Apr 20 (3 years, 4months) and the testing set was from May 20 to Feb 21 (10 months). 

# 3 EDA
[Trends, seasonality, anomalies](https://notast.netlify.app/post/2021-06-03-hierarchical-forecasting-of-hospital-admissions/), [lags and correlation of time series features and statistics](https://notast.netlify.app/post/2021-06-05-hierarchical-forecasting-of-hospital-admissions-eda-part-2/) were explored and analysed. 

- In general, there was an increase in the number of admissions till first half of 2020 during the peak of the Covid pandemic. After the peak, admissions to KTPH and NTFGH did not increase to pre peak numbers.
- The number of admissions to SKH markedly increase during 2018 as the new hospital fully opened its entire hospital campus.
<img src="https://github.com/notast/hierarchical-forecasting/blob/main/images/1%20Trend%20Hospital.png" width="500"/>
<img src="https://github.com/notast/hierarchical-forecasting/blob/main/images/1%20Trend%20Cluster.png" width="500"/>

- There are fewer admissions in Feb, likely for two reasons. Firstly, Feb has the shortest month and Chinese Lunar New Year tends to happen during Feb. 
- There are more admissions in the final quarter of the year, mostly from Oct and Dec. 
<img src="https://github.com/notast/hierarchical-forecasting/blob/main/images/1%20Season%20Hospital.png" width="550"/>

- Most of the anomaly detected occurred during the peak of COVID19 pandemic from Jan 20- Jul 20. 
- The anomaly in 2018 came from SKH and was not observed in other hospitals nor at a more aggregated cluster level. The anomaly was likely due to the change in the number of admissions before and after the hospital was opened in Jul 18.
<img src="https://github.com/notast/hierarchical-forecasting/blob/main/images/1%20Anomaly.png" width="500"/>

- Correlation of the [48 time series features and statistics](https://fabletools.tidyverts.org/reference/features_by_tag.html) was conducted as 48 is a number of variables to analyse. The correlation also determines the associative relationship between the features. Correlation was done for each level as well as a collection of all levels because admissions at superordinate levels would have some correlation with admissions at the subordinate level.
- The spread in the correlations revealed the heterogeneity of the features, the features identify a variety of time series traits.
<img src="https://github.com/notast/hierarchical-forecasting/blob/main/images/2%20Correlatio.png" width="550"/>

- PCA was conducted as most features have moderate correlation with each other and to condense the information.
- [The first 5 principal component captured 88% of the variance.](https://notast.netlify.app/post/2021-06-05-hierarchical-forecasting-of-hospital-admissions-eda-part-2/#variation-captured) 
<img src="https://github.com/notast/hierarchical-forecasting/blob/main/images/2%20PCA.png" width="450"/>

# 4 Approach (classical) 
For the classical approach, [3 hierarchical forecasting techniques were used](https://notast.netlify.app/post/2021-06-09-hierarchical-forecasting-of-hospital-admissions-classical-forecast/#reconciliation):
1. bottoms up `bu`
2. reconciliation using ordinary least square `ols`
3. reconciliation using minimum trace with sample covariance `mint_cov`

[Base models for the above techniques included](https://notast.netlify.app/post/2021-06-09-hierarchical-forecasting-of-hospital-admissions-classical-forecast/#models):
1. ETS
2. ARIMA
3. ARIMA with Covid (peak period) as regressor
4. ARIMA with Covid regressor with 1 month lag
5. ARIMA with Covid regressor with 2 month lag
6. ARIMA with Covid regressor with 3 month lag

[The best ARIMA model class was selected using `AICc`.The best ARIMA model was with a dummy variable for Covid peak period as a regressor `ARIMA(Admission ~ Covid)`](https://notast.netlify.app/post/2021-06-09-hierarchical-forecasting-of-hospital-admissions-classical-forecast/#arima).

[The best ARIMA model and ETS model were then evaluated against the testing set. The best classical model was an ARIMA model with an external regressor for Covid without any lags `ARIMA(Admission ~ Covid)` as the base and the forecast reconciled using minimum trace `mint_cov`. Across all levels, the average `rmse` was 847 and `mae` was 745](https://notast.netlify.app/post/2021-06-09-hierarchical-forecasting-of-hospital-admissions-classical-forecast/#arima-vs-ets).

## 4.1 Forecast (classical model on testing set)
[The best model's hierarchical forecast on the testing set is plotted below](https://notast.netlify.app/post/2021-06-09-hierarchical-forecasting-of-hospital-admissions-classical-forecast/#performance-for-each-level): 

Hospital level:

<img src="https://github.com/notast/hierarchical-forecasting/blob/main/images/3%20Classical%20hospital.png" width="550"/>

Cluster level:

<img src="https://github.com/notast/hierarchical-forecasting/blob/main/images/3%20Classical%20cluster.png" width="550"/>

National level:

<img src="https://github.com/notast/hierarchical-forecasting/blob/main/images/3%20Classical%20national.png" width="550"/>

# 5 Approach (machine learning) 
## 5.1 Pre-processing 
[Different combinations of predictors and engineered features were screened to determine the best combination for machine learning](https://notast.netlify.app/post/2021-06-12-hierarchical-forecasting-of-hospital-admissions-ml-approach-screen-variables/#pre-processing-recipes).

1. Basic recipe `rec_basic`
- Lags
- Rolling lags
- Covid peak period (dummy variable)
- Relevant temporal features from `step_timeseries_signature` e.g. month, year, quarter of the year 
- Hierarchical levels e.g. National level, Cluster level  
- Members in the corresponding level e.g. CGH hospital, SHS cluster 
2. Basic recipe + Time series features and statistics `rec_ft`
3. Basic recipe + PCA of the time series features and statistics `rec_PC`
4. Basic recipe + kernel PCA of the time series and features and statistics `rec_kPC` 

[Random forest model](https://notast.netlify.app/post/2021-06-12-hierarchical-forecasting-of-hospital-admissions-ml-approach-screen-variables/#modelling) [with cross-validation](https://notast.netlify.app/post/2021-06-12-hierarchical-forecasting-of-hospital-admissions-ml-approach-screen-variables/#evaluate-with-cross-validation) was used to screen the recipes. The best recipe `rec_PC` was used for machine learning modelling.
| Recipe         | RMSE (avg cv) |
|----------------|-------------|
| rec_PC         | 514         |
| rec_kPC        | 516         |
| rec_basic      | 526         |
| rec_rf         | 543         |

## 5.2 Models
The best recipe was [passed into the following models and tuned with resampling](https://notast.netlify.app/post/2021-06-14-hierarchical-forecasting-of-hospital-admissions-ml-approach-modeltime-package/#modelling):   

1. Elastic net regression with splines `GLM` 
2. Multivariate adaptive regression spline `MARS` 
3. Random forest `RF`
4. Extreme gradient boost `XGB`
5. Boosted PROPHET `PB` 
6. ~~LightGBM (LightGBM has seen success with hierarchical time series in the M5 competition but fatal errors were encountered when running it in `R`)~~

| Model                                   | RMSE (avg cv) | MAE (avg cv)  |
|-----------------------------------------|------|------|
| RF                       | 549  | 409  |
| PB                       | 1137 | 799  |
| XGB               | 1231 | 888  |
| MARS | 3796 | 3312 |
| GLM                             | 9847 | 8281 |

The top 2 models, RF and PB, were [manually retuned](https://notast.netlify.app/post/2021-06-20-hierarchical-forecasting-of-hospital-admissions-ml-approach-ensemble/#tune-again).

*Example of identifying more appropriate parameter range for retuning Prophet Boost*

<img src="https://github.com/notast/hierarchical-forecasting/blob/main/images/6%20Retuning%20PB.png" width="450"/>

Both Random Forest and Prophet Boost [benefited from retuning.](https://notast.netlify.app/post/2021-06-20-hierarchical-forecasting-of-hospital-admissions-ml-approach-ensemble/#performance-after-retuning) 
| Model                                        | RMSE (avg cv) | MAE (avg cv) |
|----------------------------------------------|------|-----|
| RF with retuning  | 545  | 409 |
| RF                            | 549  | 409 |
| PB retuning  | 945  | 673 |
| PB                           | 1137 | 799 |

## 5.3 Ensemble model 
[An ensemble model was assembled from the two top performing models, Random Forest and Prophet Boost. Both tuned and original Random Forests were trial in the ensemble as the performance between the models were minimal. The retuned Prophet Boost was nominated to be the default Prophet Boost for the ensemble as its accuracy was markedly better than the original version. As Random Forest performed much better than Prophet Boost, the weightage given to Random Forest was at least 80%.](https://notast.netlify.app/post/2021-06-20-hierarchical-forecasting-of-hospital-admissions-ml-approach-ensemble/#ensemble) 

- All the ensemble models performed better than its member models. 
- Better performing ensemble models had a stronger bias to Random Forest. 
- Some of the classical approaches performed better than the top 2 machine learning models. 

| Approach         | Model                                                                                   | RMSE (training set) | MAE (training set) |
|------------------|-----------------------------------------------------------------------------------------|------|------|
| Machine Learning | Ensemble model (RF retuned + PB retuned, weights 9:1)                                  | 535  | 412  |
| Machine Learning | Ensemble model (RF  + PB retuned,   weights 9:1)                                       | 538  | 412  |
| Machine Learning | Ensemble model (RF retuned + PB retuned, weights 8:2)                                  | 539  | 421  |
| Machine Learning | Ensemble model (RF + PB retuned, weights 8:2)                                          | 543  | 423  |
| Machine Learning | RF retuned                                              | 545  | 409  |
| Machine Learning |  RF                                                                      | 549  | 409  |
| Classical        | Reconciliation with `mint_cov`. Base model: ARIMA + Covid regressor  | 847  | 745  |
| Machine Learning | PB retuned                                              | 945  | 673  |
| Classical        | Reconcilation with `OLS`. Base model: ARIMA + Covid   regressor       | 1085 | 937  |
| Classical        | Base model of ARIMA + Covid    regressor                                                | 1117 | 991  |
| Machine Learning | PB                                                                      | 1137 | 799  |

## 5.4 Forecast (ML model on testing set)
The [best machine learning model forecast on the testing is plotted below](https://notast.netlify.app/post/2021-06-20-hierarchical-forecasting-of-hospital-admissions-ml-approach-ensemble/#performance-individual-levels):

Hospital level: 

<img src="https://github.com/notast/hierarchical-forecasting/blob/main/images/6%20Testing%20hospital.png" width="500"/>

Cluster level: 

<img src="https://github.com/notast/hierarchical-forecasting/blob/main/images/6%20Testing%20Cluster.png" width="500"/>

National level: 

<img src="https://github.com/notast/hierarchical-forecasting/blob/main/images/6%20Testing%20national.png" width="500"/>

# 6 Forecast (best model on future period)
To recap, the training set was from Jan 16 to Apr 20 (3 years, 4months) and the testing set was from May 20 to Feb 21 (10 months) and the forecast horizon was from Mar 21- Dec 21 (10 months). The best model was an ensemble model of retuned Random Forest and retuned Prophet Boost with a 9:1 weightage. [Below are the forecasted future admissions using the best model](https://notast.netlify.app/post/2021-06-20-hierarchical-forecasting-of-hospital-admissions-ml-approach-ensemble/#the-future). 

Hospital level: 

<img src="https://github.com/notast/hierarchical-forecasting/blob/main/images/6%20Future%20Hosptial.png" width="600"/>

Cluster level: 

<img src="https://github.com/notast/hierarchical-forecasting/blob/main/images/6%20Future%20Cluster.png" width="550"/>

National level: 

<img src="https://github.com/notast/hierarchical-forecasting/blob/main/images/6%20Future%20National.png" width="550"/>

# 7 Reflecting points
Most of the hospital level forecast were relatively flat; perhaps, due to insufficient data compared to cluster and national level which had more observations from the aggregation of subordinate levels. The forecast for cluster and national level appeared more plausible with some peaks and dips with an upward trend. Nonetheless, forecasting during this Covid period is challenging. Any forecast can be thrown off the rails as the situation is erratic and dynamic. For instance, the Covid infection rate was stable after Aug 20 but became [more serious in May 21 with the Singapore government implementing stricter social distancing measures](https://www.gov.sg/article/additional-restrictions-under-phase-2--heightened-alert).

# 8 Future work
- More machine learning models and deep learning
- Replacing XGB in Prophet Boost with other tree-based boost models like Catboost or lightGBM
- Predicting all hospital admission at once with a global model.  











