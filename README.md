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

Note that at this point, this is not quite working but it is close:
```script
$ docker exec -it slurm-base-root /bin/bash

root@linux:~# sacct -vvvvv -n -o jobid,jobname,user,state,elapsed,exitcode
sacct: Jobs Eligible in the time window from Thu Jan 11 00:00:00 2024 to Thu Jan 11 11:48:58 2024
sacct: debug:  Options selected:
        opt_completion=no
        opt_dup=no
        opt_field_list=jobid,jobname,user,state,elapsed,exitcode,
        opt_help=0
        opt_no_steps=no
        opt_whole_hetjob=(null)
sacct: debug3: Trying to load plugin /usr/lib/x86_64-linux-gnu/slurm-wlm/accounting_storage_slurmdbd.so
sacct: accounting_storage/slurmdbd: init: Accounting storage SLURMDBD plugin loaded
sacct: debug3: Success.
sacct: debug3: Trying to load plugin /usr/lib/x86_64-linux-gnu/slurm-wlm/auth_munge.so
sacct: debug:  auth/munge: init: Munge authentication plugin loaded
sacct: debug3: Success.
sacct: error: _slurm_persist_recv_msg: read of fd 3 failed: No error
sacct: error: _slurm_persist_recv_msg: only read 108 of 2616 bytes
sacct: error: slurm_persist_conn_open: No response to persist_init
sacct: error: Sending PersistInit msg: No error
sacct: debug2: Clusters requested:      cluster
sacct: debug2: Userids requested:       all
sacct: error: _slurm_persist_recv_msg: read of fd 3 failed: No error
sacct: error: _slurm_persist_recv_msg: only read 108 of 2616 bytes
sacct: error: Sending PersistInit msg: No error
sacct: error: DBD_GET_JOBS_COND failure: Unspecified error
```

So close...
