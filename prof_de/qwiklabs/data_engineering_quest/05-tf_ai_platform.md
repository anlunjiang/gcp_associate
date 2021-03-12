# Predict Housing Prices with Tensorflow and AI Platform

* tf.estimator — a high-level TensorFlow API that greatly simplifies machine learning programming. Estimators encapsulate the following actions:
    * training
    * evaluation
    * prediction
    * export for serving

![tf](../images/ts_hier.PNG =800x)

* Tensorflow is a hierarchical framework. The further down the hierarchy you go, the more flexibility you have, but that more code you have to write. A best practice is to start at the highest level of abstraction. Then if you need additional flexibility for some reason drop down one layer. 

* Feature columns are your Estimator's data "interface." 
    * They tell the estimator in what format they should expect data  (is it one-hot? sparse? dense? continuous?). 
    * https://www.tensorflow.org/api_docs/python/tf/feature_column

```python 
FEATURES = ["CRIM", "ZN", "INDUS", "NOX", "RM",
            "AGE", "DIS", "TAX", "PTRATIO"]
LABEL = "MEDV"
feature_cols = [tf.feature_column.numeric_column(k)
                  for k in FEATURES] #list of Feature Columns
```

* An Estimator is what actually implements your training, eval and prediction loops. Every estimator has the following methods:
    * fit() for training
    * eval() for evaluation
    * predict() for prediction
    * export_savedmodel() for writing model state to disk

```bash
%%bash
JOBNAME=housing_$(date -u +%y%m%d_%H%M%S)

gcloud ai-platform jobs submit training $JOBNAME \
   --region=$REGION \
   --module-name=trainer.task \
   --package-path=./trainer \
   --job-dir=$GCS_BUCKET/$JOBNAME/ \
   --runtime-version 1.15 \
   -- \
   --output_dir=$GCS_BUCKET/$JOBNAME/output
```

* Because we are using the TF Estimators interface, distributed computing just works! The only change we need to make to run in a distributed fashion is to add the --scale-tier argument. Cloud AI Platform then takes care of distributing the training across devices for you!

* You can also train on gpus - each gpu is 3 ML units - default project quota is 10 ML Units

# Get Predictions
* There are two flavors of the AI Platform Prediction Service: Batch and online.
    * Online prediction is more appropriate for latency sensitive requests as results are returned quickly and synchronously.
    * Batch prediction is more appropriate for large prediction requests that you only need to run a few times a day.

The prediction services expect prediction requests in standard JSON format so first we will create a JSON file with a couple of housing records.

* !gcloud ai-platform predict --model housing_prices --json-instances records.json
    * You can use your trained model as an api