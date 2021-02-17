# Predicting Visitor Purchases with a Classification Model with BigQuery ML

Flattening arrays into tables
To convert an ARRAY into a set of rows, also known as "flattening," use the UNNEST operator. UNNEST takes an ARRAY and returns a table with a single row for each element in the ARRAY.

Because UNNEST destroys the order of the ARRAY elements, you may wish to restore order to the table. To do so, use the optional WITH OFFSET clause to return an additional column with the offset for each array element, then use the ORDER BY clause to order the rows by their offset.

Example


SELECT *
FROM UNNEST(['foo', 'bar', 'baz', 'qux', 'corge', 'garply', 'waldo', 'fred'])
  AS element
WITH OFFSET AS offset
ORDER BY offset;

|----------|--------|
| element  | offset |
|----------|--------|
| foo      | 0      |
| bar      | 1      |
| baz      | 2      |
| qux      | 3      |
| corge    | 4      |
| garply   | 5      |
| waldo    | 6      |
| fred     | 7      |



SELECT
  * EXCEPT(fullVisitorId)
FROM
                    
This is pretty cool above


# SAMPLE create model from ML sql 

CREATE OR REPLACE MODEL `ecommerce.classification_model`
OPTIONS
(
model_type='logistic_reg',
labels = ['will_buy_on_return_visit']
)
AS

```sql
#standardSQL
SELECT
  * EXCEPT(fullVisitorId)
FROM

  # features
  (SELECT
    fullVisitorId,
    IFNULL(totals.bounces, 0) AS bounces,
    IFNULL(totals.timeOnSite, 0) AS time_on_site
  FROM
    `data-to-insights.ecommerce.web_analytics`
  WHERE
    totals.newVisits = 1
    AND date BETWEEN '20160801' AND '20170430') # train on first 9 months
  JOIN
  (SELECT
    fullvisitorid,
    IF(COUNTIF(totals.transactions > 0 AND totals.newVisits IS NULL) > 0, 1, 0) AS will_buy_on_return_visit
  FROM
      `data-to-insights.ecommerce.web_analytics`
  GROUP BY fullvisitorid)
  USING (fullVisitorId)
;
```

For classification problems in ML, you want to minimize the False Positive Rate (predict that the user will return and purchase and they don't) and maximize the True Positive Rate (predict that the user will return and purchase and they do).

This relationship is visualized with a ROC (Receiver Operating Characteristic) curve like the one shown here, where you try to maximize the area under the curve or AUC

![External and Internal Load Balancers](../images/roc.PNG =500x)
