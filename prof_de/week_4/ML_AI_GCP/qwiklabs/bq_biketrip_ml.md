# Predict Bike Trip Duration with a Regression Model in BQML

* BigQuery is Google's fully managed, NoOps, low cost analytics database

* BigQuery Machine Learning (BQML, product in beta) is a new feature in BigQuery where data analysts can create, train, evaluate, and predict with machine learning models with minimal coding.

* You can create models with the training data:
```sql
CREATE OR REPLACE MODEL
  bike_model.model_bucketized
OPTIONS
  (input_label_cols=['duration'],
    model_type='linear_reg') AS
SELECT
  duration,
  start_station_name,
IF
  (EXTRACT(dayofweek
    FROM
      start_date) BETWEEN 2 AND 6,
    'weekday',
    'weekend') AS dayofweek,
  ML.BUCKETIZE(EXTRACT(hour
    FROM
      start_date),
    [5, 10, 17]) AS hourofday
FROM
  `bigquery-public-data`.london_bicycles.cycle_hire
  ```

  ```sql
  SELECT * FROM ML.EVALUATE(MODEL `bike_model.model`)
  ```

| Row | mean_absolute_error | mean_squared_error | mean_squared_log_error | median_absolute_error | r2_score            | explained_variance | 
|-----|---------------------|--------------------|------------------------|-----------------------|---------------------|--------------------|
|   1 | 1025.592602254746   |1.8624065011167914E8|0.8635022545602298      | 542.2390594711619     |0.003624978745990215 | 0.0036440493634728455


* Our best model contains several data transformations.
    * Wouldn’t it be nice if BigQuery could remember the sets of transformations we did at the time of training and automatically apply them at the time of prediction? 
    * It can, using the TRANSFORM clause!
* The transformations are saved and carried out on the provided raw data to create input features for the model.

```SQL
CREATE OR REPLACE MODEL
  bike_model.model_bucketized TRANSFORM(* EXCEPT(start_date),
  IF
    (EXTRACT(dayofweek
      FROM
        start_date) BETWEEN 2 AND 6,
      'weekday',
      'weekend') AS dayofweek,
    ML.BUCKETIZE(EXTRACT(HOUR
      FROM
        start_date),
      [5, 10, 17]) AS hourofday )
OPTIONS
  (input_label_cols=['duration'],
    model_type='linear_reg') AS
SELECT
  duration,
  start_station_name,
  start_date
FROM
  `bigquery-public-data`.london_bicycles.cycle_hire
```

* With the TRANSFORM clause in place, enter this query to predict the duration of a rental from Park Lane right now
```sql
SELECT
  *
FROM
  ML.PREDICT(MODEL bike_model.model_bucketized,
    (
    SELECT
      'Park Lane , Hyde Park' AS start_station_name,
      CURRENT_TIMESTAMP() AS start_date) )
```


* A linear regression model predicts the output as a weighted sum of its inputs
    * SELECT * FROM ML.WEIGHTS(MODEL bike_model.model_bucketized)
* Note, numeric features get a single weight, while categorical features get a weight for each possible value