oxiscripts
==========
[![Ansible Lint](https://github.com/oxivanisher/role-oxiscripts/actions/workflows/ansible-lint.yml/badge.svg)](https://github.com/oxivanisher/role-oxiscripts/actions/workflows/ansible-lint.yml)

Install and configure [oxiscripts](https://github.com/oxivanisher/oxiscripts).

Per default, oxiscripts will create a backup of system files (lots of things can be configured in `/etc/oxiscripts/jobs/`) to `/mnt/backup`. When `oxiscripts_rsync_path` is set, it will automatically create a rsync job using the `oxiscripts_rsync_*` options for `/mnt/backup`.

Role Variables
--------------

| Name                                | Comment                                                                                                | Default value                     |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------ | --------------------------------- |
| oxiscripts_update                   | Upgrade oxiscripts. Be aware that this might not be 100% idempotent, so it is defaulted to be `false`. | `False`                           |
| oxiscripts_mirror                   | URL to the oxiscripts installer script.                                                                | `https://oxi.ch/files/install.sh` |
| oxiscripts_email                    | Email address where notifications are sent to.                                                         | `root@example.lan`                |
| oxiscripts_rsync_server             | Rsync server for backups                                                                               |                                   |
| oxiscripts_rsync_user               | Rsync user for backups                                                                                 |                                   |
| oxiscripts_rsync_password           | Rsync password for backups                                                                             |                                   |
| oxiscripts_rsync_path               | Rsync path for backups                                                                                 |                                   |
| oxiscripts_rsync_mnt_backup_options | Rsync options only for `/mnt/backup/` rsync backup                                                     |                                   |
| oxiscripts_rsync_documents_options  | Rsync options for the `rdiff-backup` aka `backup-documents` rsync backups. See notes below!            | `--delete`                        |
| oxiscripts_backup_cleanup           | After how many day should backups be cleaned up                                                        | `60`                              |
| oxiscripts_documents_backup         | List of backup targets for rdiff-backups, see the definition below                                     | `[]`                              |
| oxiscripts_rsync_backup             | List of backup targets for rsync backups, see the definition below                                     | `[]`                              |

The lists of dicts for `oxiscripts_documents_backup` containing the following fields:
| Name     | Comment                                                                                                                       | Default value | Example value       |
| -------- | ----------------------------------------------------------------------------------------------------------------------------- | ------------- | ------------------- |
| path     | The path to be backed up                                                                                                      | ``            | `/opt/myapp/mydata` |
| exclude  | A space separated list of folders to be excluded                                                                              | ``            | `lost+found temp`   |
| target   | The target subfolder under which the rdiff backup will be stored. The default location is `/mnt/backup/oxirdiffbackup/TARGET` | ``            | `myapp_data`        |
| duration | How long should the files locally be kept before beeing cleaned                                                               | `3M`          | `6M`                |

The example above would look like this:
```yaml
oxiscripts_documents_backup:
  - path: /opt/myapp/mydata
    exclude: lost+found temp
    target: myapp_data
    duration: 6M
```

The lists of dicts `oxiscripts_rsync_backup` containing the following fields:
| Name     | Comment                                          | Default value                     | Example value                  |
| -------- | ------------------------------------------------ | --------------------------------- | ------------------------------ |
| path     | The path to be backed up                         | ``                                | `/opt/myapp/mydata`            |
| exclude  | A space separated list of folders to be excluded | `lost+found`                      | `lost+found temp`              |
| target   | The rsync target server string                   | ``                                | `yournas::backup/myapp_backup` |
| options  | Options for rsync separated py spaces            | ``                                | `--delete`                     |
| user     | Username for the rsync server                    | `{{ oxiscripts_rsync_user }}`     | `backup_user`                  |
| password | Password for the rsync server                    | `{{ oxiscripts_rsync_password }}` | `super secret`                 |

The example above would look like this:
```yaml
oxiscripts_documents_backup:
  - path: /opt/myapp/mydata
    exclude: lost+found temp
    target: yournas::backup/myapp_backup
    options: --delete
    user: backup_user
    password: super secret
```


* To update oxiscripts, you need to set the `oxiscripts_update` variable to true. For example `ansible-playbook site.yml all --extra-vars oxiscripts_update=true`

* Example for `oxiscripts_documents_backup`:
  ```yaml
  - EXCLUDED_DIR="" rdiffbackup /srv/service_a/ factorio-server 1M
  ```
  This backups `/srv/service_a/` to the rsync server locally and finally to the rsync server into the directory `factorio-server`. Backups older than 1 month are removed.

* Example for `oxiscripts_rsync_backup`:
  ```yaml
  - EXCLUDED_DIR="lost+found" RSYNC_PASSWORD="{{ oxiscripts_rsync_password }}" rsyncbackup /srv/service_b/ backup@{{ oxiscripts_rsync_server }}::rsync-backup/service_b "--delete"
  ```
  This backups `/srv/service_b/` to the rsync server into the directory `rsync-backup/service_b`. Files no lonver available locally will be deleted.

Notes
-----

Since `rsync-backup` is now checking the source directory better for consistency, it is no longer working to just rsync over without the `--delete` option. The nice thing was, that you could have a bigger history of backups on the remote system, but you had to clean that up manually, so it was never a perfect solution. To accommodate to the new situation, the `oxiscripts/oxirdiffbackup` folder is now excluded from the default rsync backup (which has `--delete` not enabled per default) and create a custom job entry for every `backup-documents` (aka rdiff-backup) job. It will not just sync the complete `oxirdiffbackup` folder with `--delete` since it would delete no longer active backups on the target server.
Please be aware that you have to remove the corresponding entry manually from the `/etc/oxiscripts/jobs/backup-rsync.sh` file if you remove a `backup-documets` job.

Example Playbook
----------------

```yaml
- name: Install and configure oxiscripts
  hosts: all
  collections:
    - oxivanisher.linux_base
  roles:
    - role: oxivanisher.linux_base.oxiscripts
      tags:
        - oxiscripts
```
License
-------

BSD

Author Information
------------------

This role is part of the [oxivanisher.linux_base](https://galaxy.ansible.com/ui/repo/published/oxivanisher/linux_base/) collection, and the source for that is located on [github](https://github.com/oxivanisher/collection-linux_base).
