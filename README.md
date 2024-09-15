# restic_b2

Very basic ansible role for configuring and automating backups using [restic](https://restic.net/) with [Backblaze B2](https://www.backblaze.com/docs/cloud-storage-integrate-restic-with-backblaze-b2)

## Usage

Most tunable variables are defined in [defaults](defaults/main.yml).

### B2

This role uses the [b2 command-line tool](https://github.com/Backblaze/B2_Command_Line_Tool) to create a b2 bucket and application keys on the target host. The b2 cli tool is downloaded from the github releases page, and will be verified using a checksum. If you change the value of `b2_cli_version` you will also need to update `b2_cli_checksum` or the download will fail.

You will need to provide b2 application keys which have permission to create buckets and application keys using the variables `b2_application_key_id` and `b2_application_key`. These will be used with `b2` commands, but will not be stored on the host. The role will create application keys for the bucket.

b2 buckets and keys have the following default values:

```yaml
b2_bucket_name: "{{ inventory_hostname_short }}-backup"
b2_key_name: "{{ inventory_hostname_short }}-restic"
```

If set the value of `b2_resource_prefix` will be prepended to the names of the b2 bucket & keys that the role creates (for example if you want to identify buckets that belong to a specific organization or environment).

#### B2 Vars

| name                 | type  | description                                                                                 | default                         |
|----------------------|-------|---------------------------------------------------------------------------------------------|---------------------------------|
| `b2_resource_prefix` | `str` | String to prepend to the name of b2 resources                                               | `None`                          |
| `b2_bucket_name`     | `str` | Name to use for the b2 bucket                                                               | [defaults](./defaults/main.yml) |
| `b2_key_name`        | `str` | Name to use for the b2 account key                                                          | [defaults](./defaults/main.yml) |
| `b2_region`          | `str` | B2 [data region](https://www.backblaze.com/docs/cloud-storage-data-regions) for the account | `us-west-001`                   |
| `b2_cli_version`     | `str` | Version of the B2 cli to install                                                            | `4.1.0`                         |
| `b2_cli_checksum`    | `str` | Checksum for validating B2 downloads                                                        | [defaults](./defaults/main.yml) |

### Restic

Restic is installed via the system's package manager. This will work on most Linux distros, but will require some additional tooling to work more generally.

The `restic.service` and `restic.timer` configuration files are created to automate the backup process. The `restic_env_file` contains environment variable definitions used for configuring `restic` at runtime. The `restic_password_file` contains a password used by restic to secure backups. The value of the restic password needs to be provided to the role at runtime.

#### Restic Vars

| name                     | type     | description                                                                                                           | default                           |
|--------------------------|----------|-----------------------------------------------------------------------------------------------------------------------|-----------------------------------|
| `restic_bin`             | `str`    | Path to the restic binary                                                                                             | `/usr/bin/restic`                 |
| `restic_config_dir`      | `str`    | Directory where restic config files are stored                                                                        | `/etc/restic`                     |
| `restic_password_file`   | `str`    | Path where the restic password file will be written                                                                   | `/etc/restic/password`            |
| `restic_env_file`        | `str`    | Path where the restic env file will be written                                                                        | `/etc/restic/restic.env`          |
| `restic_backup_folders`  | `list`   | List of paths for restic to backup                                                                                    | `['/home']`                       |
| `restic_backup_schedule` | `string` | [systemd calendar event](https://man.archlinux.org/man/systemd.time.7#CALENDAR_EVENTS) definition for the backup time | `*-*-* 02:00:00"` (2am every day) |
