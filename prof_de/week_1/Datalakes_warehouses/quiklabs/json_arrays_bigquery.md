# Working with JSON and Array data in BigQuery

#standardSQL
SELECT
['raspberry', 'blackberry', 'strawberry', 'cherry', 1234567] AS fruit_array

You can query arrays directly and big query visualisation will flatten them automatically, but you have to make sure that the array elements are all of the same data type 

SELECT
  fullVisitorId,
  date,
  ARRAY_AGG(DISTINCT v2ProductName) AS products_viewed,
  ARRAY_LENGTH(ARRAY_AGG(DISTINCT v2ProductName)) AS distinct_products_viewed,
  ARRAY_AGG(DISTINCT pageTitle) AS pages_viewed,
  ARRAY_LENGTH(ARRAY_AGG(DISTINCT pageTitle)) AS distinct_pages_viewed
  FROM `data-to-insights.ecommerce.all_sessions`
WHERE visitId = 1501570398
GROUP BY fullVisitorId, date
ORDER BY date

You can do some pretty useful things with arrays like:

finding the number of elements with ARRAY_LENGTH(<array>)
deduplicating elements with ARRAY_AGG(DISTINCT <field>)
ordering elements with ARRAY_AGG(<field> ORDER BY <field>)
limiting ARRAY_AGG(<field> LIMIT 5)

SELECT DISTINCT
  visitId,
  h.page.pageTitle
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`,
UNNEST(hits) AS h
WHERE visitId = 1501570398
LIMIT 10

UNNEST arrays to query them indicidually 


#standardSQL
SELECT STRUCT("Rudisha" as name, [23.4, 26.3, 26.4, 26.1] as splits) AS runner

Creating structs


#standardSQL
SELECT race, participants.name
FROM racing.race_results
CROSS JOIN
race_results.participants # full STRUCT name

Cross join with the structs to essentially group by 


