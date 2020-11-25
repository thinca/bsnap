# bsnap

![](https://img.shields.io/badge/version-v0.0.1-blue)

A small wrapper to create snapshots of Btrfs.

WARNING: This is very experimental.

- Config file base.
- Create a snapshot.
- Cleanup snapshots with leaving some latest snapshots.


## Install

Put `bin/bsnap` to your `$PATH`.  This is just a Bash script.

Less dependency.  You need `btrfs` command only, maybe.


## Usage

### Create a snapshot

```shell
$ bsnap -n daily -t / -s /.snapshots create
Create a readonly snapshot of '/' in '/.snapshots/snapshot-2020-10-10T16:05:36-daily'
```

This command will create a snapshot of `/` to the place like `/.snapshpts/snapshot-2020-10-09T18:19:20-daily`.
The number part is datetime a snapshot created.


These arguments can be supplied by a config file.
Put a config file to `/etc/bsnap/root-daily`, then:

```shell
$ bsnap -c root-daily create
```

You can add minimum comment string to a snapshot.

```shell
$ bsnap -c root-daily create some-comment
Create a readonly snapshot of '/' in '/.snapshots/snapshot-2020-10-10T16:05:36-daily.some-comment'
```


### List snapshots

`list` sub command shows list of snapshots.

```shell
$ bsnap -c root-daily list
2020-10-08T04:00:00-daily
2020-10-09T04:00:00-daily
2020-10-10T04:00:00-daily
2020-10-11T04:00:00-daily
2020-10-12T04:00:00-daily
```


### Delete a snapshot

`delete` sub command deletes a snapshot.

```shell
$ bsnap -c root-daily delete
1) 2020-10-08T04:00:00-daily
2) 2020-10-09T04:00:00-daily
3) 2020-10-10T04:00:00-daily
4) 2020-10-11T04:00:00-daily
5) 2020-10-12T04:00:00-daily
Which one to delete? (C-d to cancel) > 3
Delete subvolume (no-commit): '/.snapshots/snapshot-2020-10-10T04:00:00-daily'

$ bsnap -c root-daily list
2020-10-08T04:00:00-daily
2020-10-09T04:00:00-daily
2020-10-11T04:00:00-daily
2020-10-12T04:00:00-daily
```

You can pass a snapshot name directly as an argument.

```shell
$ bsnap -c root-daily delete 2020-10-09T04:00:00-daily
Delete subvolume (no-commit): '/.snapshots/snapshot-2020-10-09T04:00:00-daily'

$ bsnap -c root-daily list
2020-10-08T04:00:00-daily
2020-10-11T04:00:00-daily
2020-10-12T04:00:00-daily
```

### Cleanup snapshots

`cleanup` sub command deletes snapshots, keeping the latest `n` count.

```shell
$ bsnap -c root-daily list
2020-10-08T04:00:00-daily
2020-10-09T04:00:00-daily
2020-10-10T04:00:00-daily
2020-10-11T04:00:00-daily
2020-10-12T04:00:00-daily

$ bsnap -c root-dialy -l 3 cleanup
Delete subvolume (no-commit): '/.snapshots/snapshot-2020-10-08T04:00:00-daily'
Delete subvolume (no-commit): '/.snapshots/snapshot-2020-10-09T04:00:00-daily'

$ bsnap -c root-daily list
2020-10-10T04:00:00-daily
2020-10-11T04:00:00-daily
2020-10-12T04:00:00-daily
```


## Config file

A config file is just a bash script.
Sets some shell variables.

```bash
# Name of snapshot
name=daily

# A directory to place snapshots
snapshots_dir=/.snapshots

# A target of snapshot
target=/

# Count of snapshots to leave on cleanup
leave_count=30

# Executes cleanup when create a snapshot
auto_cleanup=yes
```

A config file can put under `/etc/bsnap/` directory or `$HOME/.config/bsnap/` directory, and specify by `-c` option.

```shell
# There is /etc/bsnap/root-daily file:
$ bsnap -c root-daily create
```


## Restore

`bsnap` has no feature to restore from snapshot.
You can copy files from snapshot manually, or replace subvolume with snapshot manually.


## Examples

### Works with systemd-timer

Creates a service:

```systemd
# /etc/systemd/system/bsnap-create@.service

[Unit]
Description=Create a snapshot by bsnap

[Service]
Type=oneshot
ExecStart=/path/to/bsnap -c %i create

[Install]
WantedBy=default.target
```

Creates a timer:

```systemd
# /etc/systemd/system/bsnap-create-daily@.timer

[Unit]
Description=Create a snapshot by bsnap in daily

[Timer]
OnCalendar=04:00
Unit=bsnap-create@%i.service

[Install]
WantedBy=timers.target
```

Starting the timer:

```shell
$ systemctl start bsnap-create-daily@daily.timer
```


## License

[Zlib License](LICENSE.txt)


## Author

thinca <thinca@gmail.com>
