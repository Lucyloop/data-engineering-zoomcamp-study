# MAC


## 1. Running Postgres with Docker
```bash
docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:13
```

## 2. CLI for Postgres

```bash
pip install pgcli
```
Using `pgcli` to connect to Postgres

```bash
pgcli -h localhost -p 5432 -u root -d ny_taxi
```

## 3. Upload data to database in jupyter notebook
1. python create sql scheme: sqlalchemy -> pd.io.sql.get_schema
2. if dataset is too big, we can split, using iterator and chunk, so import data chunk by chunk
```py
pd.read_csv('yellow_tripdata_2021-01.csv', iterator=True, chunksize=100000)
```
3. use df.to_sql to insert data row by row
4. use \d tablename to check data scheme in postgres
5. In python, you can use read_sql to read result of sql query.

# 4. Connecting pgAdmin and Postgres
so, easy to manage server and database.
1. create and connect pdadmin
```bash
docker run -it \
  -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
  -e PGADMIN_DEFAULT_PASSWORD="root" \
  -p 8080:80 \
  dpage/pgadmin4
```
2. Running Postgres and pgAdmin together
Create a network

```bash
docker network create pg-network
```

Run Postgres (change the path)

```bash
docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v /Users/trishxie/data-engineering-zoomcamp-study/week_1_basics_n_setup/2_docker_sql/ny_taxi_postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  --network=pg-network \
  --name pg-database \
  postgres:13
```

3. run pdadmin in the same network
```bash
docker run -it \
  -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
  -e PGADMIN_DEFAULT_PASSWORD="root" \
  -p 8080:80 \
  --network=pg-network \
  --name pgadmin-2 \
  dpage/pgadmin4
```

4. open pdAdmin, create server and then connect to pg-database.

So, above, we know how to use docker to set up database environment, and how to connect database, and how to use python (sqlalcheme) to connect database and ingest data to database.

Then, next step, we should learn how to run all together with docker yaml file. so that the ingestion process is recycable. 

# 5. Finnally, compose everything into yaml and run together.
basically, two things, 1 is starting server, 2 is ingesting data into database
## docker to run py file - run ingestion.py . 
1. firstly, change jupyter file to py file
2. add if __name__ == '__main__': for outside server to run py file as a module import. 
    1. In addition, argparse can be used for users to input relevant information while running py file.
    2. use python **.py -> to run python file, to ingest data. But this is just for test. We can directly use dockerfile to run! 
3. op1: run dockerfile to run python file.
    1. some docker commands:  docker build -t taxi_ingest:v001 .
    ```bash
    docker build -t taxi_ingest:v001 .
    ```
    ```bash
    URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz"

    docker run -it \
    --network=pg-network \
    taxi_ingest:v001 \
        --user=root \
        --password=root \
        --host=pg-database \
        --port=5432 \
        --db=ny_taxi \
        --table_name=yellow_taxi_trips \
        --url=${URL}
    ```
## set up docker-compose.yaml file to run postgres and pdAdmin - run database service

    ```bash
    docker-compose up
    ```

    Run in detached mode:

    ```bash
    docker-compose up -d
    ```

    Shutting it down:

    ```bash
    docker-compose down
    ```

    Note: to make pgAdmin configuration persistent, create a folder `data_pgadmin`. Change its permission via

    ```bash
    sudo chown 5050:5050 data_pgadmin
    ```

    and mount it to the `/var/lib/pgadmin` folder:

    ```yaml
    services:
    pgadmin:
        image: dpage/pgadmin4
        volumes:
        - ./data_pgadmin:/var/lib/pgadmin
        ...
    ```