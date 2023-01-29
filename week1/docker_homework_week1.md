# DOCKER - HOMEWORK WEEK 1
## Question 1
```
$ docker build --help
```
## Question 2

```
$ docker run -it --entrypoint=bash python:3.9
container$ > pip list 
```

## Question 3
```
$ docker compose up
```
### docker-compose.yaml

```
services:
  pgdatabase:
    image: postgres:13
    environment: 
      - POSTGRES_USER=root
      - POSTGRES_PASSWORD=root
      - POSTGRES_DB=ny_taxi
    volumes:
      - "./ny_taxi_postgres_data:/var/lib/postgresql/data:rw"
    ports:
      - "5432:5432"
  pgadmin: 
    image: dpage/pgadmin4
    environment:
      - PGADMIN_DEFAULT_EMAIL=admin@admin.com
      - PGADMIN_DEFAULT_PASSWORD=root
    ports:
      - "8080:80"
```

``` $ python ingest_data.py host host localhost 5432 ny_data green_taxi_data.csv 
```

### ingest_data.py
```
#!/usr/bin/env python
# coding: utf-8

import pandas as pd
import argparse
import os
from sqlalchemy import create_engine
from time import time

def main(params):
    user = params.user
    password = params.password
    host = params.host
    port = params.port 
    db = params.db
    table_name = params.table_name
    csv_name = params.csv
    # csv_name = 'output.csv'

    # os.system(f"wget {url} -O {csv_name}")
    
    engine = create_engine(f'postgresql://{user}:{password}@{host}:{port}/{db}')
    
    df_iter = pd.read_csv(csv_name, iterator=True, chunksize=100000)

    df = next(df_iter)

    df.lpep_pickup_datetime = pd.to_datetime(df.lpep_pickup_datetime)
    df.lpep_dropoff_datetime = pd.to_datetime(df.lpep_dropoff_datetime)
    
    df.head(n=0).to_sql(name=table_name, con=engine, if_exists='replace')

    df.to_sql(name=table_name, con=engine, if_exists='append')

    while True:
        t_start=time()
        df=next(df_iter)
        df.lpep_pickup_datetime = pd.to_datetime(df.lpep_pickup_datetime)
        df.lpep_dropoff_datetime = pd.to_datetime(df.lpep_dropoff_datetime)
        df.to_sql(name=table_name, con=engine, if_exists='append')
        t_end=time()
        
        print('inserted another chunk..., took %.3f second' % (t_end - t_start))



if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='Ingest CSV data to Postgres.')

    # user
    # password
    # host
    # port
    # database name
    # table name
    # csv's url

    parser.add_argument('user', help='username for postgres')
    parser.add_argument('password', help='password for postgres')
    parser.add_argument('host', help='host for postgres')
    parser.add_argument('port', help='port for postgres')
    parser.add_argument('db', help='database name for postgres')
    parser.add_argument('table_name', help='name of the table where we will write the results to')
    parser.add_argument('csv', help='the csv file')
    # parser.add_argument('url', help='url of the csv file')

    args = parser.parse_args()

    main(args)

```
### SQL Query
```
SELECT COUNT(1) 
FROM 
	green_taxi_data 
WHERE 
	date(lpep_pickup_datetime) = date(lpep_dropoff_datetime) AND
	date(lpep_pickup_datetime) = '2019-01-15'

```
## Question 4

```
SELECT lpep_pickup_datetime, trip_distance
FROM 
	green_taxi_data 
WHERE 
	date(lpep_pickup_datetime) IN ('2019-01-18', 
								   '2019-01-28',
								   '2019-01-15',
								   '2019-01-10')
ORDER BY trip_distance DESC

```
## Question 5
```
SELECT COUNT(1)
  FROM green_taxi_data
WHERE 
	date(lpep_pickup_datetime) = '2019-01-01' AND
	passenger_count IN (2, 3)
```

## Question 6

```
SELECT "DOLocationID", tip_amount FROM 
	zones zon FULL OUTER JOIN green_taxi_data green
	ON zon."LocationID" = green."PULocationID"
WHERE
	zon."Zone" = 'Astoria'
ORDER BY
	green.tip_amount DESC
```

The first row of the results => "Astoria"	146	88

```
SELECT "Zone" FROM zones WHERE "LocationID" = 146
```

"Long Island City/Queens Plaza"
