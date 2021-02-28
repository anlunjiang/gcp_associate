# Smart Analytics - Machine Learning and AI on GCP

* Machine Learning = A way to get predicted insights using standard algorithms from data and make repeated decisions - it is a type of AI
    * Deep learning is a type of ML
* Algorithms once trained is a model for a specific use case

Standard use cases include:
* Detect a pattern in image - using say ResNet
* Predict the future of a time series
* Understand or transcribe human text/speech
* Quality Control - to save money

GCP provides options to do ML for people of different skill levels. 
* Tensorflow/AI Platform for advanced users
    * Tensorflow playground to visualise neural nets 
* Cloud AutoML - Bring your own data - and create custom models with minimal code 
* Pre-trained ML Models - Ready to go - Vision API, Speech API, Translation API, NLP, Video Intelligence API

# Pre-built ML model APIs for unstructured data
 For common ML tasks, consider Pretrained APIs

 * you can process unstructured data by labelling it with AI
    * sentiment, location etc

* AI Platform Notebooks
    * Uses the latest open source version of Jupyter Lab - can change hardware as well
    * They are treated as compute engine resources 
    * Integrated with git and connectors to bq

* You can execute BQ commands from AI platform notebooks using 

    %%bigquery df
    SELECT * FROM `bigquery-public-data.austin_bikeshare.bikeshare_trips` LIMIT 50

    Straight from the notebook and saves from bq to a pandas dataframe

* Cloud AI Platform is a fully managed service for custom ML mopdels
    * Scales to production
    * Hyperparameter tuning
    * Serverless - selftuning - manages overhead
    * JupyterLab notebooks

## KubeFlow

Google's opensource framework for building ML Pipelines. Kubeflow Pipelines package multi-step ML workflows for Kubernetes
* UI for tracking jobs and scheduling
* Engine for scheduling multi step ML workloads
* SDK - defining and manipulating pipelines
* Notebooks for interacting with the system and using the SDK

* Pipelines contain components: ML steps that are assembled into a graph that describes the execution pattern 
    * Reusable and portable
    * Not locked in to just GCP and can run on different providers
* Leverage M8s containers to solve problems
* if you have a K8s cluster - then you can run Kubeflow 
* Benefits are similar to Cloud Composer but tailored for ML workloads

* There is a detailed UI like Dagster UI - for visualising nodes and training details/data details
    * every run is logged with all config params for audit purposes 
* You can share and package pipelines easily as zips between cloud providors 

### AI Hub
 * Is a repository of AI assets - you can find prebuilt and optimised ML pipelines that can fit your needs 
    * you can have public and restricted assets as well

* Use ML on GCP using either
    * AI Platform (your model, your data)
    * AutoML (our models, your data)
    * Perception API (our models, our data)
* Use Kubeflow to deploy end-to-end ML pipelines
    * Don’t reinvent the wheel for your ML pipeline! Leverage pipelines on AI Hub

# BigQuery ML

```SQL
-- Create / Train
CREATE OR REPLACE MODEL `bqml_tutorial_sample_model`
OPTIONS(model_type=`logistic_reg`) AS SELECT training.data

-- Evaluate - splits training data and reports evaluation statistics on the held out set 
-- Precision - low false positive rate
-- Recall - ratio of correctly predicted positive observations to all observations
-- Accuracy - true postives + true negatives / entire dataset
-- Confusion Matrix can visualise these metrics 
FROM ML.EVALUATE(
    MODEL `bqml_tutorial_sample_model`, TABLE eval_table
)

-- Predict / Classify
FROM ML.PREDICT(
    MODEL `bqml_tutorial_sample_model`, table game_to_predict
) AS predict

```

Types of Models to use in bq - you can specify the type in `model_type` parameter
* Classification
    * Logistic Regression
    * DNN Deep Neural Network 
    * xgboost - gradient boosted trees 
* Linear Regression
    * DNN Regression
    * xgboost Regression
* Recommendation Engine
    * Matrix factorisation models
        * Collaborative filtering algorithms

* Note that you can also train on TensorFlow and then predict on BigQuery - by changing `model_type` as tensorflow

## Unsupervised ML with Clustering
* Supervised is when your data has labels and you know the answer
* Unsupervised has no labelled data

Unsupervised ML is about exploring patterns in our data - clustering for example

![bqml](./ML_AI_GCP/images/bqml.PNG =600x)


# Cloud AutoML

A service on GCP that allows you to build ML models with minimal effort and expertise - good for getting a model off the ground asap

* AutoML uses Googles Models but your data - intermediate between Pretrained models like Cloud Vision API compared to AI Platform
* Follows a standard procedure that is divided into :
    * Train - dataset prepared to train the model - default is 80 train 20 test (val, test 10% each)
        * Basic checks are made to determine if there is enough info and if organised properly (enough rows and adequate labels)
        * Training occurs through an entire dataset many times to iteratively get better through each epoch
        * Test group is evalutated against and removes bias towards training data 
    * Deploy 
        * Models are deployed once trained - and are autodeleted after a time - you need to retrain periodically 
    * Serve
        * Serve models using WebUI or from CURL commands - api formats
* Many models can feed into another in a hierarchical decision tree where nodes are models 

* Recommendation is to use prebuilt stuff first before trying to build your own - AutoML can actually be used with the pre-built services or on their own

## AutoML Vision
* For training models for image classification
    * Have to be base64 encoded and no more than 30MB in size for each image
    * JPEG,PNG and GIF files only 

## AutoML NLP
* For training models for text - which must be standard text
    * Deleted if unused for 60 days up to 6 months
        * Must be periodically retrained as improvements are constant so retraining is ideal for better performance
* Remove low frequency labels and increase document variety for good performance and avoid overfitting

## AutoML Tables
Is for structured data - not public information yet - but can perform ML on tabular data
* easiest way to import data to ML Tables is through BigQuery or CSVs and Cloud Storage
    * bq is good as it supports arrays and structs 
* you can allocate a budget to a model to cap training time
    * Note that AutoML will automatically stop training is the model isnt getting better


* Results that are too good to be true are usually due to data issues and overfitting
    * metrics like ROC curves, RMSE etc can determine this 

![bqml](./ML_AI_GCP/images/choose_models.PNG =800x)

Best practice is go from simple to complex first and see the results 
