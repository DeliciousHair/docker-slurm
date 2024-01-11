# Slurm in Docker

This is very stripped-down version of the original repo, the purpose is
to connect a `MySQL` container for accounting data and get the `slurmdbd`
daemon running.

## run
```script
$ docker compose build
$ docker compose up
```

Leaving the instance attached makes it easier to monitor communications
between the slurm container and the database container.

## WIP
At this point, this is not quite working but it is close:
```script
$ docker exec -it slurm-base-root /bin/bash

root@linux:~# ps fax
    PID TTY      STAT   TIME COMMAND
    119 pts/1    Ss     0:00 /bin/bash
    127 pts/1    R+     0:00  \_ ps fax
      1 pts/0    Ss     0:00 /bin/sh -c /etc/startup.sh ; /bin/bash -l
     49 ?        Sl     0:00 /usr/sbin/munged
     68 ?        Sl     0:00 /usr/sbin/slurmd
    107 ?        Sl     0:00 /usr/sbin/slurmdbd
    110 pts/0    S+     0:00 /bin/bash -l
```

Note that for whatever reason, `slurmctld` dies fast and silently, casing
`sacct` (and everything else, really) to not work.
