# Predict Visitor Purchases with a Classification Model in BQML

* BigQuery Machine Learning (BQML, product in beta) is a new feature in BigQuery where data analysts can create, train, evaluate, and predict with machine learning models with minimal coding

```sql
#standardSQL
WITH visitors AS(
SELECT
COUNT(DISTINCT fullVisitorId) AS total_visitors
FROM `data-to-insights.ecommerce.web_analytics`
),

purchasers AS(
SELECT
COUNT(DISTINCT fullVisitorId) AS total_purchasers
FROM `data-to-insights.ecommerce.web_analytics`
WHERE totals.transactions IS NOT NULL
)

SELECT
  total_visitors,
  total_purchasers,
  total_purchasers / total_visitors AS conversion_rate
FROM visitors, purchasers
```

Bigquery can query two subqueries at once!


* The web analytics dataset has nested and repeated fields like ARRAYS which need to broken apart into separate rows in your dataset. This is accomplished by using the UNNEST() function, which you can see in the above query.

![roc](../images/rocauc.PNG =500x)

* add `warm_start = true` to your model options if you are retraining new data on an existing model for faster training times.
    *  Note that you cannot change the feature columns (this would necessitate a new model).
