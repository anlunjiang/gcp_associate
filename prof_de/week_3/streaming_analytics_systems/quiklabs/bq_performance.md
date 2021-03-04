# Optimizing your BigQuery Queries for Performance

* Most of the overhead of simple queries is caused by I/O, not by computation.
* do not select * 
    *  The exception is when you use a SELECT * in a subquery, then only reference a few fields in an outer query; the BigQuery optimizer will be smart enough to only read the columns that are absolutely required.
* Cached results are extremely fast and do not incur charges.
    * whitespaces can cause a cache miss.
    * Queries are never cached if they exhibit non-deterministic behavior (for example, they use CURRENT_TIMESTAMP or RAND)
* One way to improve the read performance and avoid joins is to give up on storing data efficiently, and instead add redundant copies of data. This is called denormalization.
* Avoid self-joins of large tables - Use a window function instead of a self-join
* Reduce data being joined
* Limiting large sorts

```sql
SELECT
  rental_id,
  ROW_NUMBER() OVER(ORDER BY end_date) AS rental_number
FROM
  `bigquery-public-data.london_bicycles.cycle_hire`
ORDER BY
  rental_number ASC
LIMIT
  5

```

It takes 34.5 seconds to process just 372 MB because it needs to sort the entirety of the London bicycles dataset on a single worker. Had we processed a larger dataset, it would have overwhelmed that worker.

We might want to consider whether it is possible to limit the large sorts and distribute them. Indeed, it is possible to extract the date from the rentals and then sort trips within each day:

```sql
WITH
  rentals_on_day AS (
  SELECT
    rental_id,
    end_date,
    EXTRACT(DATE
    FROM
      end_date) AS rental_date
  FROM
    `bigquery-public-data.london_bicycles.cycle_hire` )
SELECT
  rental_id,
  rental_date,
  ROW_NUMBER() OVER(PARTITION BY rental_date ORDER BY end_date) AS rental_number_on_day
FROM
  rentals_on_day
ORDER BY
  rental_date ASC,
  rental_number_on_day ASC
LIMIT
  5
  ```
sorting can be done on just a single day of data at a time.

# Data skew
The same problem of overwhelming a worker (in this case, overwhelm the memory of the worker) can happen during an ARRAY_AGG with GROUP BY if one of the keys is much more common than the others.

# Approximate aggregation functions
BigQuery provides fast, low-memory approximations of aggregate functions. Instead of using COUNT(DISTINCT …), we can use APPROX_COUNT_DISTINCT on large data streams when a small statistical uncertainty in the result is tolerable.