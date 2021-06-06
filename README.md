# 1 Summary
The aim of the project is to forecast monthly admissions to Singapore public acute adult hospitals. The admissions were treated as a hierarchical time series. Admissions were forecasted at each level. Every country has a hierarchical order to its public hospitals. In Singapore, there are three levels:

National level
<br>
|-- Cluster level (Clusters are a network of hospitals based on geographical regions. There are 3 health clusters in Singapore.)
 <br>
|---- Hospital level (There are 8 public acute adult hospitals.)

Two forecasting approaches were taken; classical and machine learning. While [no approach is superior](https://cbergmeir.com/talks/FFDS_ACML2020.pdf), machine learning approaches reign supreme in this project but with higher computational costs and longer analysis time. 

| Approach         | Model                                                                                   | RMSE | MAE  |
|------------------|-----------------------------------------------------------------------------------------|------|------|
| Machine Learning | Ensemble model (RF retuned + PB retuned, weights 9:1)                                  | 535  | 412  |
| Machine Learning | Ensemble model (RF  + PB retuned,   weights 9:1)                                       | 538  | 412  |
| Machine Learning | Ensemble model (RF retuned + PB retuned, weights 8:2)                                  | 539  | 421  |
| Machine Learning | Ensemble model (RF + PB retuned, weights 8:2)                                          | 543  | 423  |
| Machine Learning | Random Forest with retuning (RF retuned)                                              | 545  | 409  |
| Machine Learning | Random Forest (RF)                                                                      | 549  | 409  |
| Classical        | Minimum Trace Reconciliation (MINT). Base model: ARIMA + Covid regressor  | 847  | 745  |
| Machine Learning | Prophet Boost with retuning (PB retuned)                                              | 945  | 673  |
| Classical        | Ordinary Least Square Reconciliation (OLS). Base model: ARIMA + Covid   regressor       | 1085 | 937  |
| Classical        | Base model of ARIMA + Covid    regressor                                                | 1117 | 991  |
| Machine Learning | Prophet Boost (PB)                                                                      | 1137 | 799  |
| Classical        | Bottoms up (BU). Base model: ARIMA + Covid regressor                                    | 1142 | 1043 |
| Machine Learning | Extreme Gradient Boosting                                                               | 1231 | 888  |
| Classical        | Base model of ETS                                                                       | 1951 | 1788 |
| Classical        | OLS. Base model: ETS                                                                    | 2193 | 2028 |
| Classical        | BU. Base model: ETS                                                                     | 2343 | 2146 |
| Classical        | MINT. Base model: ETS                                                                   | 3045 | 2814 |
| Machine Learning | Multivariate Adaptive Regression Spline                                                 | 3796 | 3312 |
| Machine Learning | Elastic Net                                                                             | 9847 | 8281 |
| Machine Learning | LightGBM (unable to run in my `R`)                                                      | NA   | NA   |

Forecasting admissions at hospital levels can help hospital managers plan for better manpower deployment during predicted peak periods. Forecasting at higher levels such as cluster of hospitals or even national level can help senior management and policy planners develop better plans to deal with high and low key periods and review other strategies to reduce the admission rate. A manageable admission rate helps to ensure clinicians will have sufficient time to review their patients.  

The datasets, model outputs and key objects are housed on this GitHub. The rest of the README outlines the project, more details are found in [my blog](https://notast.netlify.app/tags/modeltime/)
# 2 Data
The [dataset is monthly admission to Singapore public acute adult hospitals](https://www.tablebuilder.singstat.gov.sg/publicfacing/mainMenu.action). The [dataset starts from Jan 2016 and ends in Feb 2021](https://github.com/notast/hierarchical-forecasting/blob/main/stat_sg.csv). The forecast horizon was 10months, i.e. to forecast till the end of 2021 (Mar 21- Dec 21). The training set was from Jan 16 to Apr 20 (3 years, 4months) and the test set was from May 20 to Feb 21 (10 months). 

# 3 EDA


# 4 Approach (classical) 

# 5 Approach (machine learning) 

## 5.1 Pre-processing 

## 5.2 Models

## 5.3 Ensemble model 

# 6 Future Work
