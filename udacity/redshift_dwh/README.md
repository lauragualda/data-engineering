## Analytics Database for Sparkify

In this project we created a new database for the Analytics team at Sparkify. 
The analytics team is particularly interested in understanding what songs users are listening to, but was not able to easily access the app output data.
For that, we gathered data from user logs and songs metadata from JSON files and designed a schema in Postgres to match their specific use case.

The schema is based on the fact table `songplays` contanining high level information of each song listened by each user. From this table, analysts can understand aggregated user behavior, such as: the number of songs a user listens on a day, how many different songs an user listens on a time range. We also defined 4 dimension tables: `users`, `songs`, `artists`and `time`. 

### Structure of this repository

This repository contains the following files: 

1. Three python scripts used to maintain the Sparkify database:
  * `sql_queries.py` - Collection of SQL queries to drop, create and insert records into the staging tables and DWH databases
  * `create_tables.py` - Sequential commands to create staging tables and DWH tables
  * `etl.py` - ETL processes that extract user logs and songs metadata from s3, loads into staging tables on Redshift and loads relevant data into the 5 DWH tables described above
2. A config file containing information about:
  * the Amazon Redshift endpoint 
  * IAM role
  * s3 bucket containing JSON files
  * AWS key and secret (deleted for safety reasons)

### How to run the scripts

Download or clone this repository and make sure you run the following commands from inside your local directory where you saved it.

1. Add your AWS key and secret to dwh.cfg

2. Make sure the Redshift cluster is active and all the info related to it on the config is correct

3. Create the Sparkify database with its 5 tables:

```python
python create_tables.py
```

4. Extract data from s3, stage into Redshift and load it into the Sparkify DWH tables:

```python
python etl.py
```

### Analytics use cases

When joining the tables available on Sparkify, analysts will be able to easily explore the profile of users that listen to given songs or artists, look for clusters of songs and artists listened by given profiles of users and estimate which time of the day or the week users are more likely to be listening to music.

For example, an analysis on which day of the week users listen to more songs can start with the following query:

```sql
SELECT user_id, weekday, count(session_id) as n_sessions 
FROM songplays 
JOIN time USING (start_time) 
GROUP BY 1, 2
ORDER BY 1, 2 
```

And if we look at the results for a given user, like the one with `user_id=15`, we can observe that the two occasions when he listened to more than 100 songs in a day were Wednesdays (weekday 2). 

| user_id | weekday | n_sessions |
|---------|---------|------------|
| 15      | 4       | 74         |
| 15      | 5       | 27         |
| 15      | 2       | 101        |
| 15      | 4       | 9          |
| 15      | 1       | 2          |
| 15      | 2       | 18         |
| 15      | 3       | 5          |
| 15      | 0       | 10         |
| 15      | 1       | 59         |
| 15      | 2       | 103        |
| 15      | 3       | 33         |
| 15      | 0       | 22         |

Well, does that make any sense? We can now investigate looking at all the data :)
