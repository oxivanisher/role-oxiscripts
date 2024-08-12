oxiscripts
==========

Install and configure [oxiscripts](https://github.com/oxivanisher/oxiscripts).

Role Variables
--------------

| Name          | Comment                              | Default value |
|---------------|--------------------------------------|---------------|
| oxiscripts_update | Upgrade oxiscripts. Be aware that this might not be 100% idempotent, so it is defaulted to be `false`. | `False`          |
| oxiscripts_mirror | URL to the oxiscripts installer script. | `https://oxi.ch/files/install.sh`          |
| oxiscripts_email  | Email address where notifications are sent to. | `root@example.lan`          |
| oxiscripts_rsync_server | Rsync server for backups |          |
| oxiscripts_rsync_user  | Rsync user for backups |          |
| oxiscripts_rsync_password | Rsync password for backups |           |
| oxiscripts_rsync_path | Rsync path for backups |           |
| oxiscripts_backup_cleanup | After how many day should beckups be cleaned up | `60` |
| oxiscripts_documents_backup  | List of backup targets for rdiff-backups | `[]`          |
| oxiscripts_rsync_backup | List of backup targets for rsync backups | `[]`          |

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
