# BigQuery in Jupyter Labs on AI Platform

You can directly run pip isntall and big query statements within a jupyter notebook within cloud AI platform

e.g. 

```sql
%%bigquery df
SELECT
  departure_delay,
  COUNT(1) AS num_flights,
  APPROX_QUANTILES(arrival_delay, 10) AS arrival_delay_deciles
FROM
  `bigquery-samples.airline_ontime_data.flights`
GROUP BY
  departure_delay
HAVING
  num_flights > 100
ORDER BY
  departure_delay ASC
  ```



