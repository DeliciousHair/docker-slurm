# Slurm in Docker

This is very stripped-down version of the original repo, the purpose is
to connect a `MySQL` container for accounting data and get the `slurmdbd`
daemon running.

## run
```script
$ docker compose up
```

Leaving the instance attached makes it easier to monitor communications
between the slurm container and the database container.

