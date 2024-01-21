### Analyzing Developer Activity and Collaboration using Airbyte Quickstarts with Dagster, BigQuery, Google Colab, dbt, and Terraform
![image](https://github.com/btkcodedev/community_posts/assets/47392334/5e3b6e78-efde-4a77-8c9d-a319aa4b0f42)

**_Airbyte could be used as a wonderful tool to leverage the power of data with useful transformations._**
**_This transformed data could be further employed to train AI models (examples provided at the end)_**
 
In this tutorial, the GitHub API is used as the source, and streams are transformed to gain insights into developer activity, such as commits over time, top collaborators, etc. This information can be fed into an AI model for training, thereby enhancing its prediction capabilities.

**I've made a full code walk-through at [Google colab notebook](https://colab.research.google.com/drive/14U7NYK4dy5fBN3891Tbkl3SJYEqxkMYr?usp=sharing)**

You could either download it as ipynb and run with local jupyter or run step-by-step with local cmd
(If hyperlink is broken, try: https://colab.research.google.com/drive/14U7NYK4dy5fBN3891Tbkl3SJYEqxkMYr?usp=sharing)

The initial part of setting up Airbyte for pulling data from GitHub source to BigQuery and SQL transformations are already given precisely at [quickstarts directory](https://github.com/airbytehq/quickstarts/blob/main/developer_productivity_analytics_github/README.md)

<u>**Architecture**</u>

![Architecture](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/i5ttnw2eoa9ybmz8uv20.png)

<u>Explanation:</u>

Tech stacks: Dagster, dbt, Airbyte, GitHub API, BigQuery, Terraform

_We are intended to pull data from GitHub source via Airbyte user interface towards BigQuery dataset, The Airbyte UI is automated via Terraform Provider._

_After the dataset creation, data build tool (dbt) is used for transforming data with SQL queries for various metric findings viz. average time per PR, mean of total commits etc..._

## Part 1.1: Setting Up the Data Pipeline

Screenshots are attached to the colab notebook, 

_Airbyte and GitHub API:_

The GitHub source connector requires three credentials:
1. Repository name
2. GitHub personal access token
3. Workspace ID

_Airbyte and BigQuery:_ 

The BigQuery destination connector requires three credentials:
1. IAM & Admin service account JSON key
2. Google cloud project ID
3. BigQuery Dataset ID

Ref: [Configuration steps](https://github.com/airbytehq/quickstarts/blob/8268d1b01ad2f8cfcff0a75c0bd4c0c9a45d197d/developer_productivity_analytics_github/README.md?plain=1#L120)

In case you are wondering about behind the scenes, refer to [GitHub](https://github.com/airbytehq/quickstarts/blob/main/developer_productivity_analytics_github/infra/airbyte/main.tf), where you could see the sync between GitHub and BigQuery

![Sync](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nq3xp665x1npljg4s7b5.png)

You could find the reference of number of streams at GitHub
![GitHub](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/12o4f5uc5j3gmpgl0bsq.png)

After running `terraform apply`, Terraform jobs will commence. Subsequently, the Airbyte UI will be set up and ready with all the configuration details, and the streams will be prepared for pulling.

## Part 1.2: Transformations with dbt

Setup the environment variables used for dbt setup, Currently three 
1. [dbt_service_account_JSON_key](https://github.com/airbytehq/quickstarts/blob/8268d1b01ad2f8cfcff0a75c0bd4c0c9a45d197d/developer_productivity_analytics_github/dbt_project/profiles.yml#L8)
2. [bigquery_project_id](https://github.com/airbytehq/quickstarts/blob/8268d1b01ad2f8cfcff0a75c0bd4c0c9a45d197d/developer_productivity_analytics_github/dbt_project/profiles.yml#L13)
3. [github_source.yml](https://github.com/airbytehq/quickstarts/blob/8268d1b01ad2f8cfcff0a75c0bd4c0c9a45d197d/developer_productivity_analytics_github/dbt_project/models/sources/github_source.yml#L6)

Either setup those env variables or hard code in the files for next steps.
Run `dbt debug` for confirming the setup.

The schema for table population for each stream could be seen at [GitHub](https://github.com/airbytehq/quickstarts/tree/8268d1b01ad2f8cfcff0a75c0bd4c0c9a45d197d/developer_productivity_analytics_github/dbt_project/models/staging)
![GitHub](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/85t7t2dhkjym2y50shxi.png)

After running `dbt run --full-refresh`, You could find the transformed tables populated in the BigQuery dataset 

![BigQuery Dataset](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gss195f5l6c7zf9pjis2.png)

_The [dbt marts](https://github.com/airbytehq/quickstarts/blob/8268d1b01ad2f8cfcff0a75c0bd4c0c9a45d197d/developer_productivity_analytics_github/dbt_project/models/marts/dev_activity_by_day_of_week_analysis.sql) are very useful where insights are extracted from the pulled data and could be further utilized for AI training purposes (*Provided large dataset)_

![dbt marts](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jzea9c8sva8svbtywzrk.png)

## Part 1.3: Orchestration using Dagster

_Dagster and BigQuery:_

Dagster is a modern data orchestrator designed to help you build, test, and monitor your data workflows.

After running `dagster dev`, localport would be opened or dagster where the workflow could be seen and syncs could be monitored

![Dagster](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/g9ifronpvg9j3dbp740d.png)

## Part 1.4: Future Reference AI Model Creation in Colab ft. Tensorflow

_Export BigQuery data to Colab:_

1. Use the BigQuery connector in Colab to load the desired data from your analysis tables.
2. Preprocess the data by cleaning, filtering, and transforming it for your specific model inputs.
3. Build a Tensorflow Model for Team Dynamics and Productivity:

Specific code example is provided at [Colab](https://colab.research.google.com/drive/14U7NYK4dy5fBN3891Tbkl3SJYEqxkMYr#scrollTo=1l9f-FNihyMr)


![Scikit learn model](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bg2yb905xlv2auqi7ki8.png)

- Another example of Tensorflow Model:
  - Choose a suitable architecture like LSTM or RNN for time series analysis of developer activity, or use scikit-learn for quantitative analysis.
  - Train the model on historical data, using features like time to merge PRs, commits per day, code review frequency, etc.
  - Evaluate the model performance on validation data.

```
import tensorflow as tf
import numpy as np
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
import matplotlib.pyplot as plt

# Load data from BigQuery
client = bigquery.Client()
query = """
SELECT
  author.email,
  author.time_sec AS time_sec,
  committer.email,
  committer.time_sec AS time_sec_1,
  committer.time_sec - author.time_sec AS time_difference
FROM
  `micro-company-task-367016.transformed_data.stg_commits`,
  UNNEST(difference) AS difference
WHERE
  TIMESTAMP_SECONDS(author.time_sec) BETWEEN TIMESTAMP("2023-01-01") AND TIMESTAMP("2023-12-31")
LIMIT 1000
"""
data = client.query(query).to_dataframe()

# Preprocess data
features = ['time_sec', 'time_sec_1']
target = 'time_difference'

# Drop rows with missing values
data = data.dropna(subset=features + [target])

# Split the data into training and testing sets
X_train, X_test, y_train, y_test = train_test_split(data[features], data[target], test_size=0.2, random_state=42)

# Standardize features
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# Define TensorFlow model architecture
model = Sequential([
    Dense(32, activation='relu', input_shape=(len(features),)),
    Dense(16, activation='relu'),
    Dense(1)  # Output layer, no activation for regression
])

# Compile the model
model.compile(optimizer='adam', loss='mean_squared_error')

# Convert y_train to a NumPy array with a compatible dtype
y_train_np = y_train.values.astype('float32')

scaler_y = StandardScaler()
y_train_scaled = scaler_y.fit_transform(y_train_np.reshape(-1, 1))
y_test_scaled = scaler_y.transform(y_test.values.reshape(-1, 1))

# Train the model
model.fit(X_train_scaled, y_train_scaled, epochs=50, batch_size=32, validation_data=(X_test_scaled, y_test_scaled))


# Convert y_test to a NumPy array with a compatible dtype
y_test_np = y_test.values.astype('float32')

# Evaluate the model
mse = model.evaluate(X_test_scaled, y_test_np)
print(f'Mean Squared Error on Test Data: {mse}')

# Generate synthetic new data
new_data = pd.DataFrame({
    'time_sec': np.random.rand(10) * 1000,  # Adjust the range as needed
    'time_sec_1': np.random.rand(10) * 1000  # Adjust the range as needed
})

# Preprocess new data
new_data_scaled = scaler.transform(new_data[features])

# Make predictions on new data
predictions = model.predict(new_data_scaled)
predictions_inverse = scaler_y.inverse_transform(predictions)

# Display predictions 
print(predictions_inverse)


# Plot model predictions
plt.plot(predictions, label="Predicted Time Difference")
plt.plot(y_test.values, label="Actual Time Difference")
plt.xlabel("Data Point")
plt.ylabel("Time Difference")
plt.title("Predicted vs. Actual Time Difference")
plt.legend()
plt.show()

```
Results of model:

![Results](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qds1m9y2o4p9yw9qu94y.png)


- Other use cases

> - To predict future trends in team dynamics and productivity.
> - Identify factors that influence collaboration and individual developer performance.
> - Visualise the results using charts, graphs, and network visualisations.

## Final thoughts
With 350+ source connectors and extensive data warehouse destinations, Airbyte offers a significant advantage. We can leverage the power of transformed data as training data for various AI models, specifically tailored for predictions, especially in developer behavioral analysis, stock prediction, etc.

Links:
1. Colab Notebook: https://colab.research.google.com/drive/14U7NYK4dy5fBN3891Tbkl3SJYEqxkMYr?usp=sharing
2. Airbyte Quickstarts Repository: https://github.com/airbytehq/quickstarts
3. Airbyte Main Repository: 
https://github.com/airbytehq/airbyte
