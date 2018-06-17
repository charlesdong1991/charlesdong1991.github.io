This OpenAQ is a global air pollution measurement dataset on Google Cloud platform, and the purpose is to use SQL to do gain some insight on this data (https://www.kaggle.com/open-aq/openaq/data). Therefore, I wrote down some basic SQL codes to illustrate this dataset.

First, will import google cloud bigquery:
```python
import pandas as pd
from google.cloud import bigquery

client = bigquery.Client()
```
 Then, since I quickly learn from the dataset that there is no NULL values, so just take a look at some statistic metrics of this dataset:

```python 
query1 = """
SELECT avg(value) AS avg_value, min(value) AS min_value, MAX(value) AS max_value, pollutant
FROM `bigquery-public-data.openaq.global_air_quality`
GROUP BY pollutant
"""
list(client.query(query1).result(timeout=100))
```

Then i find the min values are almost all negative, thus take a look at the dataset, and assume these might be wrong values, and I check the number of rows containing these negative values, and find it can be ignored compared to the size of this dataframe. Therefore, in the later query, I all set value >= 0.

```python
query2 = """
SELECT COUNT(*) FROM `bigquery-public-data.openaq.global_air_quality`
WHERE value < 0
"""
list(client.query(query2).result(timeout=100))[0].values()
```

Check which are the most severe pollutants in the world:
```python
query3 = """
SELECT count(*) AS freq, pollutant
FROM `bigquery-public-data.openaq.global_air_quality`
WHERE value >= 0
GROUP BY pollutant
ORDER BY freq DESC
"""
list(client.query(query3).result(timeout=100))
```

We can also know in every country, which pollutant is their biggest threat. And the result is ordered in alphabetical sequence.

```python
query4 = """
SELECT f.freq, f.country, f.pollutant
FROM
(
SELECT COUNT(*) AS freq, country, pollutant, 
       ROW_NUMBER() OVER (PARTITION BY country ORDER BY COUNT(*) DESC) AS rownum
FROM `bigquery-public-data.openaq.global_air_quality`
WHERE value >= 0
GROUP BY country, pollutant
ORDER BY freq DESC
) f
WHERE f.rownum = 1
ORDER BY f.country
LIMIT 20
"""
list(client.query(query4).result(timeout=100))
```
I notice that the same city/location can have pollution records at different timestamps, we can see the average pollution value of a specific pollutant over all timestamps a city/location has. And in this example, we will see the O3 value at different places.

```python
def print_queried_cities_pollution(pollutant):
    
    client = bigquery.Client()
    query = """
        SELECT city, AVG(value) AS value, AVG(latitude) AS latitude, AVG(longitude) AS longitude
        FROM `bigquery-public-data.openaq.global_air_quality`
        WHERE value>=0 and pollutant = @pollutant
        GROUP BY city
        """
    query_params = [
        bigquery.ScalarQueryParameter('pollutant', 'STRING', pollutant)
    ]
    job_config = bigquery.QueryJobConfig()
    job_config.query_parameters = query_params
    query_job = client.query(query, job_config=job_config)

    query_job.result()
    # Alternative to print the result row-wisely.  
    destination_table = query_job.destination
    table = client.get_table(destination_table)
    for row in client.list_rows(table):
        print(row)

# check the O3 measurement in different places
print_queried_cities_pollution('o3')
```

