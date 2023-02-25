
### build and run container
```bash
docker build -t test:pandas .
```
```bash
docker run -it test:pandas 2021-01-15
```

### build postgres in a container
```dash
docker run -it \
  -e POSTGRES_USER=root \
  -e POSTGRES_PASSWORD=root \
  -e POSTGRES_DB=ny_taxi \
  -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql/data \
  -p 5432:5432
  postgres:13
```
- volumes (`-v`) : a way of mapping folder in the file system on the host machine to a folder in the container 
--> dont want to lose the state when we run the docker everytime thats why we need to map a folder on 
our host machine to a folder in the container--> called mounting 

- also need to specify the ports (`-p`): map a port on host machine to a port in container --> 
  needed to send a request to database --> need to be a specific port in postgres

### connect to postgres locally:
- install pgcli
```bash
pip install pgcli
```
- connect
```bash
pgcli -h localhost -p 5432 -u root -d ny_taxi
```

### SQL
- DDL (data definition language): 
  - a standard for commands that define the different structure in a database.
  - create, modify and remove database objects such as tables, indexes, and users.

### pgadmin --> GUI for postgres
- Run in docker
  - ```bash
    docker run -it \
      -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
      -e PGADMIN_DEFAULT_PASSWORD="root" \
      -p 8080:80 \
      dpage/pgadmin4
    ```
- if we want to run postgres and pgadmin at the same time, we have to create a network --> 
bec they were running in different containers
- Create a network
  ```bash
  docker network create pg-network
  ```
  - Run Postgres (change the path)
  ```bash
  docker run -it \
    -e POSTGRES_USER="root" \
    -e POSTGRES_PASSWORD="root" \
    -e POSTGRES_DB="ny_taxi" \
    -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql/data \
    -p 5432:5432 \
    --network=pg-network \
    --name pg-database \
    postgres:13
  ```
  - Run pgAdmin
  ```bash
  docker run -it \
    -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
    -e PGADMIN_DEFAULT_PASSWORD="root" \
    -p 8080:80 \
    --network=pg-network \
    --name pgadmin-2 \
    dpage/pgadmin4
  ```


### Dockerize a pipeline
- package `argparse`: allows to parse command line arguments
- package `psycopg2`: postgreSQL database adapter for python, allows python to access postgres
- paramaterize the ingest_data
  ```bash
  URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz" 
  python ingest_data.py \
    --user=root \
    --password=root \
    --host=localhost \
    --port=5432 \
    --db=ny_taxi \
    --table_name=yellow_taxi_trips \
    --url=${URL}
  ```
- Build the image
  ```bash
  docker build -t taxi_ingest:v001 .
  ```
  
- Run the script with Docker
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
  
### Docker-compose
- compose is a tool for defining and running multiple-container Docker applications --> 
  a way of running multiple images, put multiple services into one configuration