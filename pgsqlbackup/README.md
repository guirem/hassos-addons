# hassos-addon-pgsqlbackup

## About

This addon gives you the possibility to backup a postgresql database accessible from homeassistant.

It uses defualt postgresql tools `pg_dump` (or `pg_dumpall`) to create the archive.
Scheduled backup is available with daily cleaning of old files.


## Features

- scheduled periodic backup (cron service)
- cleaning of old backups based on archiving rules.
- manual backup via hass service call
- manual restore via hass service call


### Automatic Periodic Backups

A cron schedule is used to perform automatic backup.
https://pkg.go.dev/github.com/robfig/cron?utm_source=godoc#hdr-Predefined_schedules 

Output directory is /share/pgsqlbackup. Child folders called daily, weekly and monthly are automatically created.


## Configuration

| parameter | description |
|--|--|
| __________ | _______________________ |
| log_level | Log level (debug/info/error). Defaults to `info` |
| dump_extra_opts | Additional [options](https://www.postgresql.org/docs/12/app-pgdump.html#PG-DUMP-OPTIONS) for `pg_dump` (or `pg_dumpall` [options](https://www.postgresql.org/docs/12/app-pg-dumpall.html#id-1.9.4.13.6) if `postgres_cluster is set`). Defaults to `-Z6`. |
| backup_suffix | Filename suffix to save the backup. Defaults to `.sql.gz`. |
| __________ | _______________________ |
| dbsettings |  |
| postgres_host | Postgres connection parameter; postgres host to connect to. Required. |
| postgres_db | Comma or space separated list of postgres databases to backup. Required. |
| postgres_port | Postgres connection parameter; postgres port to connect to. Defaults to `5432`. |
| postgres_user | Postgres connection parameter; postgres user to connect with. Required. |
| postgres_password | Postgres connection parameter; postgres password to connect with. Required. |
| postgres_cluster | Set to `TRUE` in order to use `pg_dumpall` instead. Also set POSTGRES_EXTRA_OPTS to any value or empty since the default value is not compatible with `pg_dumpall`. |
| __________ | _______________________ |
| schedule |  |
| enable | Enable cron feature. Defaults to `true` |
| cron | [Cron-schedule](http://godoc.org/github.com/robfig/cron#hdr-Predefined_schedules) specifying the interval between postgres backups. Defaults to `@daily`. |
| backup_keep_days | Number of daily backups to keep before removal. Defaults to `7`. |
| backup_keep_weeks | Number of weekly backups to keep before removal. Defaults to `4`. |
| backup_keep_months | Number of monthly backups to keep before removal. Defaults to `6`. |
| __________ | _______________________ |


## Example

```yaml
log_level: info                         # optional / default=info
dbsettings:
  postgres_host: 77b2833f-timescaledb   # required
  postgres_db: mydatabase               # required
  postgres_user: postgres               # required
  postgres_password: password           # required
  postgres_port: 5432                   # optional / default=5432
  postgres_cluster: false               # optional / default=false
dump_extra_opts: '-Z6 --blobs'          # optional / default=-Z6
backup_suffix: '.sql.gz'
schedule:
  enable: true                          # optional / default=true
  cron: '@daily'                        # optional / default=daily
  backup_keep_days: 7                   # optional / default=7
  backup_keep_weeks: 3                   # optional / default=3
  backup_keep_months: 6                 # optional / default=6
```

## Service usage

Backup and restore actions can be called directly from home assistant by calling `hassio.addon_stdin` service.

### Backup database

Example to run a backup task to output /share/pgsqlbackup/backup.sql.gz.
It will take settings provided in addon configuration to connect the postgresql database instance.

Output directory is /share/pgsqlbackup.

```yaml
  service: hassio.addon_stdin
  data:
    addon: 80e24959_pgsqlbackup  
    input: 
        command: backup
        output: backup.sql.gz
        extra_args: -Z6
```


### Restore database

Example to run a restore task from input /share/pgsqlbackup/backup.sql.gz.
It will take settings provided in addon configuration to connect the postgresql database instance.

input directory is /share/pgsqlbackup.

```yaml
  service: hassio.addon_stdin
  data:
    addon: 80e24959_pgsqlbackup
    input: 
        command: restore
        input: backup.sql.gz
```