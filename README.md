# Grid Power Demand Forecasting: Using classical machine learning models  

**Jay Khandelwal**  

## 1.Summary of the process:  
As I was restricted to using only classical models, no LSTMs and autoregressive models were used.   
**Best run model:** CatBoost Regressor  
**MAPE SCORE:** 1.62%  
**RMSE:** 255.3 MW  
  
## 2.Problem Statement  
Accurate elecetricity demand forecasting is critical for power grid stability and efficient energy management. Overestimating demand leads to wasted geeration and financial lossed, while underestimating demand causes sever grid instability and load shedding.   
I was asked to build a robust machine learning pipeline to predict the short term power demand based on historical consumption, environmental factors,and economic indicators.   
Also, I was restricted to using classical machine learning models (Linear regressors or Tree based), prohibiting the use of DL architectures and autoregressive packages.  

## 3. Data Preparation and Anamoly Resolution:
The provided dataset, since it represents real world data, contained a few inconsistencies and outlying spikes and drops in data that clearly weren't possible. Thus, rather than row deletion, I used linear interpolation to replace the inconsistent readings.  
A few of those were:  
* Z-score Anamoly : The main demand_mw time series data contained physically impossible data outliers such as a recorded spike of 121,000 MW in demand, when the average throughout the years was close to 8500 MW. Rather than simply deleting the respective rows or applying a rule that might not take into account the seasonal variations, I used new parameters such as demand_over_last_week, demand_over_last_24_hours, etc. which could help the model track the various timelines to explain seasonal variations, increased demands over a specific timeline, etc.   
* Weekend Mapping : To encode the weekdays, I checked the day-wise averages over the year, and against the expectation, it turned out that the lowest averages showed up on Fridays and Saturdays. Upon further investigation, it turned out, that the data is from the company Power Grid Company of Banladesh (PGCB), which operates in Bangladesh primarily, where weekends are generally assigned to Fridays and Saturdays, thus explaining the data. So, rather than taking the conventional weekends, I used this knowledge, since on weekends, the demand would be expected to be lower, since factories major consumers are closed.
* Macroeconomic Integration : Finally, I integrated the World Bank report, forward filled the indicators snf merged the final data, aligning it all for the model to traing with a long-term baseline of the grid capacity and economic growth.

## 4. Feature Architecture & Physical Intuition

Because classical regressors evaluate rows independently, the concept of "time" had to be mathematically engineered into the tabular feature space to satisfy the project constraints. The feature architecture was designed to capture both human behavioral cycles and the physical constraints of the national grid.

* **Human Habit (The Daily & Weekly Cycles):** Analysis revealed that the strongest predictor of grid demand is human routine. Features such as 'demand_lag_24' (yesterday's demand at the exact same hour) and 'rolling_mean_168h' (the weekly baseline) were engineered to capture strict work and sleep schedules. Feature importance metrics later confirmed demand_lag_24 as the single most powerful driver in the dataset.
* **Short-Term Grid Momentum:** To allow the model to "ride the wave" of immediate events like an afternoon heatwave or a sudden evening peak, short-term memory features like 'demand_lag_1' and 'rolling_mean_6h' were constructed. This gave the model localized context regarding the grid's immediate trajectory.
* **The Rebound Effect (Grid Physics):** A highly specialized 'load_shedding_lag_1' feature was designed to capture the physical reality of forced power cuts. By giving the model the "memory" of a recent grid failure, it successfully learned to anticipate the subsequent surge of pent-up electricity demand that occurs the moment power is restored.
* **Clock vs. Weather:** Standard calendar flags ('hour', 'day_of_week') were encoded. Interestingly, feature importance analysis proved that the rigid structure of human daily schedules (hour) exerted a stronger predictive influence on the model than raw environmental factors like 'temperature`', confirming that grid load is fundamentally a human behavioral metric.

## 5. Model Evaluation
To ensure strict integrity and prevention of data leakage, I was asked to evaluate the model on a continuous test set. The primary metric used was Mean Absolute Percentage Error (MAPE).   
* **Random Forest Baseline:** Initially, I used Random Forest Regressor for a solid baseline, which achieved a great test run with MAPE of 1.95 % and RMSE of 355.15 MW.  
* **Gradient Boosting:** Then, CatBoost regressor was trained. By utilizing sequential error correction, it outperformed the baseline model, achieving a MAPE of 1.62 % and a RMSE of 257 MW.
* **The Ensembling experiment:** To try if the variations could be further stabilized, we tried ensembling by averaging the predictions of the Random Forest and CatBoost models. However, testing proves the ensemble wasn 



