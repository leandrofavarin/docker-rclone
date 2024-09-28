# docker-rclone

Docker image to perform a [rclone](http://rclone.org) sync based on a cron schedule, with [healthchecks.io](https://healthchecks.io) monitoring.

Forked from [pfidr34/docker-rclone](https://github.com/pfidr34/docker-rclone).

rclone is a command line program to sync files and directories to and from:

* Google Drive
* Amazon S3
* Openstack Swift / Rackspace cloud files / Memset Memstore
* Dropbox
* Google Cloud Storage
* Amazon Drive
* Microsoft OneDrive
* Hubic
* Backblaze B2
* Yandex Disk
* SFTP
* FTP
* HTTP
* The local filesystem

## Usage

### Configure rclone

rclone needs a configuration file where credentials to access different storage
provider are kept.

By default, this image uses a file `/config/rclone.conf` and a mounted volume may be used to keep that information persisted.

A first run of the container can help in the creation of the file, but feel free to manually create one.

```
$ mkdir config
$ docker run --rm -it -v $(pwd)/config:/config ghcr.io/leandrofavarin/docker-rclone
```

### Perform sync in a daily basis

A few environment variables allow you to customize the behavior of rclone:

* `SYNC_SRC` source location for `rclone sync/copy/move` command. Directories with spaces should be wrapped in single quotes.
* `SYNC_DEST` destination location for `rclone sync/copy/move` command. Directories with spaces should be wrapped in single quotes.
* `SYNC_OPTS` additional options for `rclone sync/copy/move` command. Defaults to `-v`
* `SYNC_OPTS_EVAL` further additional options for `rclone sync/copy/move` command. The variables and commands in the string are first interpolated like in a shell. The interpolated string is appended to SYNC_OPTS. That means '--backup-dir /old\`date -I\`' first evaluates to '--backup-dir /old2019-09-12', which is then appended to SYNC_OPTS. The evaluation happens immediately before rclone is called.
* `SYNC_ONCE` set variable to only run the sync one time and then exit the container
* `RCLONE_CMD` set variable to `sync` `copy` or `move`  when running rclone. Defaults to `sync`
* `RCLONE_DIR_CMD` set variable to `ls` or `lsf` for source directory check style. Defaults to `ls`
* `RCLONE_DIR_CMD_DEPTH` set the limit of the recursion depth to this. Defaults to `-1` (rclone default)
* `RCLONE_DIR_CHECK_SKIP` set variable to skip source directory check before sync. *Use with caution*
* `CRON` crontab schedule `0 0 * * *` to perform sync every midnight. Also supprorts cron shortcuts: `@yearly` `@monthly` `@weekly` `@daily` `@hourly`
* `CRON_ABORT` crontab schedule `0 6 * * *` to abort sync at 6am
* `FORCE_SYNC` set variable to perform a sync upon boot
* `CHECK_URL` [healthchecks.io](https://healthchecks.io) url or similar cron monitoring to perform a `GET` after a successful sync
* `FAIL_URL` Fail URL to perform a `GET` after unsuccessful execution. By default this is `CHECK_URL` with appended "/fail" at the end
* `HC_LOG` set variable to send log data to healthchecks.io. `OUTPUT_LOG` must also be set.
* `OUTPUT_LOG` set variable to output log file to /logs
* `ROTATE_LOG` set variable to delete logs older than specified days from /logs
* `TZ` set the [timezone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) to use for the cron and log `America/Chicago`
* `UID` set variable to specify user to run rclone as. Must also use GID.
* `GID` set variable to specify group to run rclone as. Must also use UID.
* `SUCCESS_CODES`: set variable to a space-separated list of return codes that are considered successful. If the return code matches any of these codes, the response will be marked as a success. Example: `SUCCESS_CODES="0 8 10"`. Default value is `0`

**When using UID/GID the config and/or logs directory must be writeable by this UID**

```bash
$ docker run --rm -it -v $(pwd)/config:/config -v /path/to/source:/source -e SYNC_SRC="/source" -e SYNC_DEST="dest:path" -e TZ="America/Chicago" -e CRON="0 0 * * *" -e CRON_ABORT="0 6 * * *" -e FORCE_SYNC=1 -e CHECK_URL=https://hchk.io/hchk_uuid ghcr.io/leandrofavarin/docker-rclone
```

See [rclone sync docs](https://rclone.org/commands/rclone_sync/) for source/dest syntax and additional options.

<br />
<br />
<br />
<br />
Credit to Brian J. Cardiff for the orginal project @ https://github.com/bcardiff/docker-rclone
