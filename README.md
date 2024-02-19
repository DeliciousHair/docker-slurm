# Slurm in Docker

This was originally a fork of [a similar project][source-repo], but it has been
altered to the point of being completely different and became it's own thing.
The purpose of this work is to provide a containerised means of accessing
existing slurm accounting data for (possibly offline) analytics work without
having to use [sacct][sacct-page] on a production system with all the possible
risks and limitations that may involve.

## Who is this for?
The scenario:
* You have been asked to do a forensic analysis of activity data--no problem.
* The activity data is shipped as a MySQL database dump--no problem.
* The database and all the data was created and populated by
  [slurm][slurm-page]--uh oh.

For mere mortals like myself, the last point is a bit of an issue since the
table structures that slurm has developed works well with [sacct][sacct-page],
but not as well for us bald apes.

This project solves this problem by providing a containerized instance of the
[slurm database daemon][slurmdbd-page] that you, dear reader, may run
[sacct][sacct-page] queries on.

Note that convenience is favoured over security and it is assumed that all steps
are being run locally or in a similarly isolated system. **DO NOT** expose any
of the containers to the world at large if you care about your data at all.
**You have been warned**.

## Usage
We assume that a MySQL dump file has been provided. Connecting this container
to an existing MySQL database as a non-root user is beyond the scope of this
work.

### Create a database
Use the official MySQL docker image to make use of the database dump first:
```script
$ docker compose -f docker-compose-db-setup.yml build
$ docker compose -f docker-compose-db-setup.yml up -d
```

This will create a database:
* **database**: `slurm_data`
* **user**: `root`
* **password**: `H3ll012Three`

Secure? Not even close! But that is OK becasue this is only for experimental
purposes, right?

Verify that the `slurm_data` database exists and is accessible by `root`. This
step is especially important if you have changed any of the above details:
```script
$ docker run -it \
>  --network docker-slurm_default \
>  --rm \
>  mysql:latest \
>  mysql -hslurm_db -Dslurm_data -uroot -pH3ll012Three
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.2.0 MySQL Community Server - GPL

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> \q
Bye
```

Fantastic! We now populate `slurm_data` with our dump data:
```script
$ docker exec -i slurm_db sh -c 'exec mysql -uroot -pH3ll012Three -Dslurm_data' < /path/to/the/database/dump
```

Depending on the size of the data and the speed of your hardware, this step
may take anywhere from a few minutes to several days or more.

Once the process completes, be sure to to shut down the container:
```script
$ docker compose -f docker-compose-db-setup.yml down
```

### Explore your data
**IF** in the previous step you decided you didn't like to be root, used a
different database name, created a different password or anything along these
lines then you will first need to update the `./build/slurm.conf` and
`./buid/slurmdbd.conf` files (at a minimum) in order to account for the
changes.

Build and start the containers:
```script
$ docker compose build
$ docker compose up -d
```

The slurm container(s) use `supervisord` in an ad-hoc manner to initialize 
the services, so it can take up to a minute or so for the services to settle
in and function correctly. Once this happens, you can run `sacct` queries on
the `slurmdbd` container. For example:
```script
$ docker exec slurmdbd-root-build \
> sacct -X -P -a -M all \
> -S 2023-04-01T00:00:00 \
> -E 2023-04-01T01:00:00 \
> -o UID,JobID,NCPUS,NNodes,Submit,End,State,ConsumedEnergy,CPUTime \
> > demo_response.psv
```

This allows you to then use your favorite analytics tools on the resulting CSV
data. For example:
```python
import polars as pl

pl.read_csv(
    "demo_response.psv",
    separator="|",
    infer_schema_length=10_000,
)

# output:
# shape: (5_974, 9)
# ┌───────┬─────────┬───────┬────────┬───┬─────────────────────┬───────────┬────────────────┬──────────────┐
# │ UID   ┆ JobID   ┆ NCPUS ┆ NNodes ┆ … ┆ End                 ┆ State     ┆ ConsumedEnergy ┆ CPUTime      │
# │ ---   ┆ ---     ┆ ---   ┆ ---    ┆   ┆ ---                 ┆ ---       ┆ ---            ┆ ---          │
# │ i64   ┆ str     ┆ i64   ┆ i64    ┆   ┆ str                 ┆ str       ┆ str            ┆ str          │
# ╞═══════╪═════════╪═══════╪════════╪═══╪═════════════════════╪═══════════╪════════════════╪══════════════╡
# │ 23676 ┆ 999836  ┆ 672   ┆ 3      ┆ … ┆ 2023-04-01T06:02:59 ┆ COMPLETED ┆ 14.50M         ┆ 57-11:05:36  │
# │ 24744 ┆ 1001275 ┆ 512   ┆ 2      ┆ … ┆ 2023-04-02T02:16:03 ┆ COMPLETED ┆ 104.67M        ┆ 474-12:01:04 │
# │ 24744 ┆ 1001276 ┆ 512   ┆ 2      ┆ … ┆ 2023-04-02T02:34:15 ┆ COMPLETED ┆ 103.95M        ┆ 480-00:25:36 │
# │ 24744 ┆ 1001278 ┆ 512   ┆ 2      ┆ … ┆ 2023-04-02T02:22:07 ┆ COMPLETED ┆ 102.13M        ┆ 475-03:31:12 │
# │ …     ┆ …       ┆ …     ┆ …      ┆ … ┆ …                   ┆ …         ┆ …              ┆ …            │
# │ 22592 ┆ 3992208 ┆ 18    ┆ 1      ┆ … ┆ 2023-04-01T01:04:38 ┆ COMPLETED ┆ 0              ┆ 01:29:06     │
# │ 22592 ┆ 3992209 ┆ 18    ┆ 1      ┆ … ┆ 2023-04-01T01:04:16 ┆ COMPLETED ┆ 0              ┆ 01:22:30     │
# │ 22592 ┆ 3992210 ┆ 18    ┆ 1      ┆ … ┆ 2023-04-01T01:04:12 ┆ COMPLETED ┆ 0              ┆ 01:21:18     │
# │ 22592 ┆ 3992211 ┆ 18    ┆ 1      ┆ … ┆ 2023-04-01T01:03:56 ┆ COMPLETED ┆ 0              ┆ 01:07:30     │
# └───────┴─────────┴───────┴────────┴───┴─────────────────────┴───────────┴────────────────┴──────────────┘
```

Your results will, of course, be different.

[sacct-page]: <https://slurm.schedmd.com/sacct.html>
[slurm-page]: <https://slurm.schedmd.com/sacct.html>
[slurmdbd-page]: <https://slurm.schedmd.com/slurmdbd.html>
[source-repo]: <https://github.com/nathan-hess/docker-slurm>
