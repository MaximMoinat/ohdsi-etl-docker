# ohdsi-etl-docker

OHDSI ETL template with Docker.

This will contain the following Docker containers:

- Data quality dashboard
- R script container
- Achilles container
- Jupyterhub (optional)
- Postgresql (with open ports)
- Dedicated ETL, to be specified in the .env
- Data dump / data import facility

## Usage

First, copy `env.template` to `.env` and fill in the necessary variables. Also add a `vocab.zip` to `data/vocabulary`.

Now you can run `docker-compose up -d` to start the stack. Jupyterhub will be available on port 8080, with any user name and password `JUPYTER_PASSWORD` from `.env`. Postgresql will be available on port 5432 with user `postgres` and password `POSTGRESQL_PASSWORD` from `.env`.

### Scripts

To _run your ETL_, specify the ETL docker image in `.env` in the `ETL_IMAGE` variable. Then run

```
bin/run-etl [docker-compose options]
```

The ETL docker image should follow the following conventions:
- Vocabulary data is present at `/data/vocabulary/vocab.zip`
- Input and/or output data is present at `/data/etl/`

Put the actual vocabulary and ETL data in `./data/vocabulary/vocab.zip` and `./data/etl/` respectively.

To _load a Synthea dataset_, run `bin/load-synthea <dataset_name> <dataset_schema> [<synthea parameters>]`. For example:

```shell
bin/load-synthea "Synthea1K" "synthea1k" -p 1000
```

This will create a Synthea dataset of 1000 people and load it into the `cdm_synthea1k` schema using name `Synthea1K`. Afterwards, it is analysed with Achilles. The original generated Synthea dataset is available in schema `synthea1k`. For more command-line parameters to Synthea, please see [Synthea command-line instructions](https://github.com/synthetichealth/synthea/wiki/Basic-Setup-and-Running). To regenerate the dataset, run `bin/generate-synthea` with synthea runtime parameters.

To _run Data Quality Dashboard_ run
```
bin/run-script -e CDM_SCHEMA=cdm_synthea1k -e RES_SCHEMA=cdm_synthea1k_results r-script -p 8090:8090 dqd
```
and when it is finished, open the browser at <http://localhost:8090>. Output data will be in the `data/r-script` directory.

To _run any other R script_ at `src/r/myscript.R`, run
```
bin/run-script r-script r /scripts/myscript.R
```

Any data put in the `/output` folder is put in `./data/r-script`.

To _export the result of your ETL_, run
```
bin/pg_dump --schema=myschema
```
The result will be stored in `data/etl.dump.gz`.

_Other helper scripts_ in the `bin/` directory have the following purpose:
- `generate-results <CDM name> <raw schema name>` lets you generate the results table and schema of a given source.
- `run-script` lets you run one of the scripts in the `docker-compose.scripts.yml` file. Examples: `bin/run-script synthea` generates a Synthea dataset, `bin/run-script r-script r /scripts/myscript.R` runs your own script in `src/r/myscript.R`.
- `run-sql <file>` lets you run the commands in a given SQL file. It does environment variable substitution, substituting the value of the `MY_ENV` variable into any `${MY_ENV}` references in the given SQL.
- `run-sql-cmd <CMD>` lets you run a single SQL command.

### Other operations

To stop the stack, run

```
docker-compose down
```

and to remove all existing data as well, run

```
docker-compose down -v
```

To completely reload the vocabulary, run
```
bin/run-sql-cmd "DROP SCHEMA vocab CASCADE"
bin/run-sql-cmd "CREATE SCHEMA vocab"
bin/start
```
