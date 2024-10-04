# Week 1 - Docker and Terraform

## Creating the Image and accessing via Terminal

The docker image used for this experiment can be activated with the following terminal command. Note that the container must be running to access later with other tools
```bash
$ docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v $PWD/ny_taxi_postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:13
```

The lines in this code perform the following functions
* `-e POSTGRES_USER="root"`: specify login user for postgres
* `-e POSTGRES_PASSWORD="root"`: specify login password for postgres
* `-e POSTGRES_DB="ny_taxi"`: specify database name for postgres
* `-v $PWD/ny_taxi_postgres_data:/var/lib/postgresql/data`: create a volume mount for storing data to access later; `$PWD` gives the current working directory in bash
* `-p 5432:5432`: specify the port for postgres
* `postgres:13`: postgres image being ran

The `postgres_container.sh` file can be also be used to run the docker image. This needs to be converted to be executable using the following command:
```bash
$ sudo chmod +x postgres_container.sh
```

You can access the `ny_taxi_postgres_data` volume mount to view the information available by entering the directory, however you may need to run the following command if permission issues are encountered
```bash
$ sudo chmod a+rwx [path_to_directory]/ny_taxi_postgres_data
```

With the container active, from a new terminal window the `pgcli` package can be used to access the database via command line. The password will need to be provided after running this command
```bash
$ pgcli -h localhost -p 5432 -u root -d ny_taxi
```

Running the following command will show all the tables available in the database. Currently there are none, so the result should be blank
```postgres
\d
```

There were initially the following dependency issues with `psycopg` when trying to connect to the db via `pgcli`
```
ImportError: no pq wrapper available.
Attempts made:
- couldn't import psycopg 'c' implementation: No module named 'psycopg_c'
- couldn't import psycopg 'binary' implementation: No module named 'psycopg_binary'
- couldn't import psycopg 'python' implementation: libpq library not found
```

These were alleviated by installing the following dependencies
```bash
$ pip3 install "psycopg[binary,pool]"
```

## Obtaining and Writing to Database using Python

Code with comments for pulling and writing the taxi data to files and then postgres can be found in `TaxiDataExploreAndWrite.ipynb`. Alternatively, this can be ran using the `DataIngestion.py` script with user input and the below command. This will be used later with containerization. Note that for either of these scripts to write data, the docker container must be running.

```bash
$ URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz"

$ python ingest_data.py \
  --user=root \
  --password=root \
  --host=localhost \
  --port=5432 \
  --db=ny_taxi \
  --table_name=yellow_taxi_trips \
  --url=${URL}
```

After loading the data via the notebook, the following should now be shown when running `\d` after opening a connection to the db with `pgcli`

```
root@localhost:ny_taxi> \d
+--------+------------------+-------+-------+
| Schema | Name             | Type  | Owner |
|--------+------------------+-------+-------|
| public | pickup_locations | table | root  |
| public | yellow_taxi_data | table | root  |
+--------+------------------+-------+-------+
```

Queries can also be made against the tables. The below gets the count of records in the `yellow_taxi_data` table

```
root@localhost:ny_taxi> select count(*) from yellow_taxi_data
+---------+
| count   |
|---------|
| 3066766 |
+---------+
```
## Docker Networks