# BiogasAssessment - By Subha Vadlamani

## Problem
To build a predictive model with data set that includes operational data from two biogas facilities

    • Facility 1
    • Facility 2
The energy output (in BTU) formula below is a major feature in the problem.
Energy Output (BTU) = Flow Rate (SCFM)×Duration (minutes)×
( Methane Percentage / 100 ) × 10¹⁰ BTU/scf
Drop or spike in energy output is critical to ensure plant efficiency. Hence a best target variable for anamoly detection in this time series data set.

## Solution Initial Approach
Fiddled a lot with the data set initially due to lack of my domain knowledge.
To understand the dataset, loaded a sample of 50000 rows on Amazon Sagemaker Auto pilot and tried to build a regression predictive model for Flow rate mentioned above. Since i experimented in free tier unable to share the XGBOOST algorithm it built. But gave me a little sense of the data.
With that initial knowledge built by exploring the dataset, i went to chatgpt and started reverse engineering by asking questions like what could possibly be target variables apart from energy output mentioned above. From there i tried building a baseline code and refactored it to make it clean using chatgpt.
Along with Amazon sagemaker auto pilot, Amazon sagemaker studio, Chatgpt, for a tiny bit explore julius.ai

## Assumptions and Limitations
All the units mentioned on the visualizations are based on my understanding of the name of column and pardon me for my mistakes if any.
Modelling is done only on sample dataset due to compute resources being unavailable. So, may not have built best model.
Since the environment i built this project is on personal AWS and credentials cannot be shared. I have put together the code and outputs of the code (ALL HTML FILES) in a github repo and sharing with you.
Excuse my tardiness for not organizing the structure of the repo.

## Data Loading & Preprocessing
Read data from S3 using awswrangler. Converted json file attributes to columns and combined data of both facilities together. Didnt get a chance to explore the geospatial analysis.
Shortened headernames by removing suffix. Renamed columns that calculate energy output to what they are mentioned in the formula for readability.
Ran data quality checks and during this process i applied something i learned from chatgpt that Flat sensors (columns with low variance or constant values) are uninformative. Correlation also not found with any other column in the dataset in the heatmap later. Hence dropped them
Standardized datatypes and computed energy output.
To eliminate noise escpecially with columns used to calculate energy output applied Rolling Median Smoothing.
Finally generated correlation heatmap.

## Exploratory and Stationarity Analysis
Done univariate and multivariate analyis. Generated various plots based on columns i felt are of importance.
Some plots on seasonal decompostion are also generated.
Also after handling missing data generated autoprofiling report from sweetviz named "sweeviz_report"
Conducted ADF tests on multiple key features and plotted below:
    Rolling statistics
    Histogram with normal distribution overlay
But code for this was commented due to low compute.

Derived insights from this analysis is:
Some features were non-stationary
Trend and seasonality present in variables like energy_output_btu

## Feature Engineering
Handled missing values. Numeric columnsare filled with median. Categorical columns are filled with mode.
Derived new features below:

    high_energy: flag for top 25% energy output
    energy_flow_ratio: energy output to flow rate
    temp_diff: oil cooler outlet – injection temp
    Created lag features (_lag1, _lag2, _lag3)
    Created rolling features: 3-period and 7-period mean and std
    Added one-hot encoding for facility and site_comm_date
    Applied PCA (top 12 components retained)
    Tried but not implemented Autoencoder based features
    
### Dimensionality Reduction
Applied PCA and first 12 components explained 95%+ variance. Added these are features to the dataset.
Generated a sample 2D PCA projection visualization.

## Feature Selection & Model Training
Samples feature engineered dataset as credits on AWS are limited.
Used Recursive Feature Elimination (RFE) and Random forest classifier to select top features.
Trained and tuned Random Forest model via GridSearch
Scored on test set using:

    Accuracy, Precision, Recall, F1, AUC

Best features are mostly derived features that played important role due to sampling : Rolling stats, Lag variables, PCA components

### Visual Evaluation
For Visual evaluation of model plotted Confusion matrix, ROC curve and Feature importance plot (top 20 ranked)

###  Sequence Modeling (LSTM)
On classified dataset above prepared sequence windows for time-series forecasting.
LSTM built for univariate prediction (e.g., energy_output_btu)
Evaluated other variables as well but didnt use them.
Extended code to support vl_comp_suction_pressure

### Anomaly Detection
Applied Isolation Forest to below variables to find anamolies:
    bge_h2soutlet_temp
    vl_comp_suction_temp
    vl_comp_suction_pressure.

Detected outliers with contamination. Created anomaly_report.csv
Visualized anomalies per feature and Filtered by last 30 days if timestamp was available.

### Auto-Generated Reports
Tried and Discarded the model evaluation sweetviz HTML report.
CSV output of anomaly details. Logs and plots inline in the notebook.

## Key Insights

Energy vs Flow Patterns observed: High energy_output_btu aligns with higher flow_rate, but there are outliers.
Temperature Differentials observed: temp_diff between cooler and injection varies significantly — may have operational impact.
Important Predictors:  As already mentioned due to sampling of data used in training, Lag and rolling features provided the most predictive power. PCA also captured useful signal.
Anomalies Detected are: Most frequent in vl_comp_suction_pressure and bge_blowersuction_temp.

Thank you for taking time to read this. Thank you!
