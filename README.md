restic-scheduler
=============================

systemd based backup schedules for [restic](https://restic.net/) with per-instance config files

## Features ##

* Configuration file for each backup task stored in  `/etc/restic/<task>.conf`
* Invoke [restic](https://restic.net/) via systemd service instance
* Backup operation executed as **unprivileged user** via [linux capabilities](https://linux.die.net/man/7/capabilities) `CAP_DAC_READ_SEARCH`
* Monthly data cleanup (forget+prune) and integrity check
* systemd `restic-scheduler@.service` to run the backup
* systemd `restic-retention@.service` to apply the data retention policy (`restic forget --prune`) and run backup integrity check
* send backup logs (from journald) via [taskrunner](https://github.com/AenonDynamics/taskrunner-sh) to custom http endpoint (optional)

## Package Installation ##

### Via Aenon-Dynamics Repository ###

See [AenonDynamics/CPR](https://github.com/AenonDynamics/CPR#debian-packages)

```
apt-get install restic-scheduler
```

### Manual setup ###

* Copy the files of `systemd/` to `/lib/systemd/system`
* Create backup user `restic`
* Create config path `/etc/restic`
* Create cache path `/var/cache/restic`

## Setup ##

### Setup systemd for scheduled backups ###

restic-scheduler comes with a systemd service and timer. You can easily modify the scripts to match your needs (backup schedule, config path).

The service file is designed to work on per-instance base and uses the instance name as config file name `/etc/restic/%I.conf`.
All options of the config file are exposed as environment variables for the restic process - you can use any [supported env vars](https://restic.readthedocs.io/en/latest/040_backup.html#environment-variables).

**Configuration**

File: `/etc/restic/<mybackup>.conf`

```conf
## full support for restic env vars
# @see https://restic.readthedocs.io/en/latest/040_backup.html#environment-variables

## repository address
# @env
RESTIC_REPOSITORY="sftp:myhost.example.org:/backupdir"

## repository password provided via file
# @env
RESTIC_PASSWORD_FILE="/etc/restic/repo.key"

## additional args
# @restic-scheduler.service
BACKUP_ARGS="--exclude-if-present .nobackup --exclude-file /etc/restic/exclude.list"

## paths to be included in the snapshot
# @restic-scheduler.service
BACKUP_PATHS="/opt /srv /home"
```

**Enable schedule**

```bash
systemctl enable restic-scheduler@mybackup.timer --now
```

**systemd files**

* `/lib/systemd/system/restic-scheduler@.timer`
* `/lib/systemd/system/restic-scheduler@.service`


### Setup data retention schedule ###

restic allows an easy setup of data rentention policies regarding a [predefined schedule](https://restic.readthedocs.io/en/latest/060_forget.html#removing-snapshots-according-to-a-policy). restic-scheduler supports the time-based schemes via config file.

To use other schemes you can override the systemd file (drop in) or create a custom systemd service.

The service file is designed to work on per-instance base and uses the instance name as config file name `/etc/restic/%I.conf`

Note: for security reasons, the policy `--keep-last` is set to 10!



**Configuration**

File: `/etc/restic/<mybackup>.conf`

```conf
# @restic-retention.service
RETENTION_POLICY_DAYS=14
RETENTION_POLICY_WEEKS=12
RETENTION_POLICY_MONTHS=18
RETENTION_POLICY_YEARS=2

## additional arguments
# @restic-retention.service
RETENTION_ARGS=""

## data validation (restic check)
# ---------------------------------

## additional arguments
# @restic-retention.service
VALIDATION_ARGS=""
```

**Enable schedule**

```bash
systemctl enable restic-retention@mybackup.timer --now
```

**Defaults**

* `/lib/systemd/system/restic-retention@.timer`
* `/lib/systemd/system/restic-retention@.service`


### Send journald log to http endpoint ###

Sometime it's useful to receive backup logs via e-mail or push-notification without the requirement of a central syslog service.
restic-scheduler comes with support for [taskrunner](https://github.com/AenonDynamics/taskrunner-sh) to log job outputs via a custom http endpoint.

In case of restic-scheduler, an additional systemd service is invoked which extracts the log output from journald as well as start/stop timestamps and the exit code.

**Enable log transmission**

```bash
systemctl enable restic-scheduler-log@mybackup.service --now
systemctl enable restic-retention-log@mybackup.service --now
```

**Defaults**

* `/lib/systemd/system/restic-scheduler-log@.service`
* `/lib/systemd/system/restic-retention-log@.service`


## Contribution ##

The **.deb** package is automatically generated via a **Continuous Delivery Pipeline** - please do not build packages manually!

## License ##
restic-scheduler is OpenSource and licensed under the Terms of [Mozilla Public License 2.0](https://opensource.org/licenses/MPL-2.0). You're welcome to [contribute](docs/CONTRIBUTING.md)