[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](http://www.gnu.org/licenses/gpl-3.0)

# ansible-role-backup

configure hosts as backup client / server and run a scheduled backup from the server using rsync+ssh

optional: handle lxc container on proxmox for backup and use lvm snapshots

## Steps:

### hosts file

add the groups `backup_client` and `backup_server` to the hosts file.

```
[backup_client]
host1
host2

[backup_server]
bsrv
```

### site.yml

add the backup role to the server and the clients

```
- hosts: backup_server
  roles:
    - { role: backup, tags: backup }
- hosts: backup_client
  roles:
    - { role: backup, tags: backup }
```

### host vars

#### backup_server

```
backup_server: yes
# optional you can adjust some paths
backup_server_path_local: /srv/backup
backup_server_path_sshkey: /root/.ssh/id_rsa
backup_server_path_logdir: /var/log/backup
backup_server_path_status: /var/tmp/backup.status

```

#### backup_client

```
# list of directories to backup
backup_client_dirs:
  - /etc/
  - /srv/

# list of pattern to use as --include= arg
backup_client_include: []

# list of pattern to use as --exclude= arg
backup_client_exclude: []

# list of services to stop befor and start after the backup
backup_client_services: []

# list of databases to dump before the backup
backup_client_databases: []

# optional: list of commands to run before backup
# backup_client_exec_pre:

# optional: list of commands to run after backup
# backup_client_exec_post

# rotate backups
backup_client_rotatecount_daily: 0
backup_client_rotatecount_weekly: 0
backup_client_rotatecount_monthly: 0
backup_client_rotate_weekly_day: 0 # 0=sunday
backup_client_rotate_monthly_day: 1 # -1=last day of month
```

optional: configure client to backup multiple lxc containers

```
backup_client_pve: {...}
# if configured, the backup client can be a backup proxy for proxmox lcx containers running on it
# q: why use this method instead of running the client script on the container itself?
# a: we can use lvm snapshots so the downtime of the services on the container can be minimized
# if you like to make use of this, ensure the backup_client_user has the right to perform the required tasks
# or use ackup_client_user: root + backup_client_user_sshdir: /root/.ssh
```

# TODOs / FIXMEs

- [ ] mysql dump: make user / pw configurable
- [ ] check mk local check
- [ ] test exclude / include args (esp. regarding the "Sending command"
- [ ] rsync -R and --delete ...

