PostgreSQL
=========

Ansible role to managed [PostgreSQL](http://www.postgresql.org/). This role used official PostgreSQL repository http://download.postgresql.org/pub/repos/ instead of version available in distribution.

This role have been tested in CentOS 7 with PostgreSQL version 9.2, 9.3, 9.4, 9.5, 9.6, 10 and 11.


Requirements
------------

The only external dependency is the python bindings for PostgreSQL, which this role will install.

Role Variables
--------------

| Variable | Default value | Descriptions |
| ---------|---------------|--------------|
| `postgresql_version` | 9.6 | PostgreSQL version to deploy. Supported version: **9.2** to **9.6**, **10** and **11** |
| `postgresql_service_name` | postgresql-{{ postgresql_version }}          | Name of postgresql service for the init      |
| `postgresql_admin_user`   | postgres             | Name of the administrative user for          |
| `postgresql_admin_group`  | `postgresql_admin_user` | Name of the administrative user for       |
| `postgresql_listen_addresses` | 0.0.0.0 |  Listen address postgresql service bind to |
| `postgresql_locale`       | en\_US.UTF-8         | Default locale                               |
| `postgresql_conf_dir`      | /etc/postgresql      | Default configuration directory |
| `postgresql_data_dir`     | /var/lib/pgsql/data  | Path to the PostgreSQL data directory        |
| `postgresql_enable_cron_backup`             | no | Whether to deploy a cron to database backups |
| `postgresql_backup_dir` | /var/backup/postgresql | Path to where database backups should be |
| `postgresql_backup_age` | 0 |  Maximum age of backups to keep in days |
| `postgresql_backup_compression` | gzip |  Compression to use for backups |

*Note*: Custom user configuration must be defined in `postgresql_conf_dir`/conf.d/02-custom.conf file which override default PostgreSQL settings.

Monitoring user options :

| Variable | Default value | Descriptions |
| ---------|---------------|--------------|
| `postgresql_monitoring_enable` | False | Boolean to enable monitoring user |
| `postgresql_monitoring_user` | centreon | PostgreSQL monitoring username |
| `postgresql_monitoring_pass` | _autogenerated_ | PostgreSQL monitoring assword |

Kernel tuning options for postgresql :

| Variable | Default value | Descriptions |
| ---------|---------------|--------------|
| `postgresql_sysctl_vm_dirty_background_ratio` | 1 | Avoid letting too much data waiting in memory (vm.dirty\_background\_ratio) |
| `postgresql_sysctl_vm_dirty_ratio` | 10 | Avoid saturation of the disk during write time (vm.dirty\_ratio) |
| `postgresql_sysctl_vm_swappiness` | 10 | Push down the swap usage, which is not good for postgresql (vm.swappiness) |
| `postgresql_sysctl_vm_zone_reclaim_mode` | 0 | This allocation mechanism is useless for postgresql (vm.zone\_reclaim\_mode) |
| `postgresql_sysctl_kernel_shmmax` | 100% of RAM | Maximum size of shared memory segment in bytes (kernel.shmmax) |
| `postgresql_sysctl_kernel_shmall` | 4194304 | Total amount of shared memory available in pages of 4K (kernel.shmall) |

PostgreSQL tuning options:

| Variable | Default value | Descriptions |
| ---------|---------------|--------------|
| `postgresql_shared_buffers_prct` | 0.25  | Shared buffers is the area where PostgreSQL keeps a cache of data blocks, contrary to MySQL, you must not put all your memory there, because PostgreSQL is intended to be used with the OS-level filesystem cache. Using too much memory here will lead to inefficient cache duplication. |
| `postgresql_shared_buffers` | 25% of RAM |  |
| `postgresql_work_mem` | 1MB | This is the amount of memory that PostgreSQL can use for sort operation, hash tables (GROUP BY...) etc. before switching to a temporary table, the default value is usually fine unless you have really big tables. In which case you might want to increase this parameter. Be careful however, because each client can allocate work\_mem more than once, so putting too much here might make your server run out of memory. |
| `postgresql_maintenance_work_mem_prct` | 0.05 | This is the amount of memory that PostgreSQL can use for heavy operations like ALTER, VACCUUM, etc... Not many of them will run simultaneously, so you can put a bigger value than `work_mem` here. |
| `postgresql_maintenance_work_mem` | 5% of RAM |  |
| `postgresql_checkpoint_segments` | 16 | The number of WALs that can be filled before a checkpoint is triggered. |
| `postgresql_effective_cache_size_prct` | 0.5 | This parameter is a bit different than others, it will not cause any memory allocation, it is only used to help the query planner by telling how much memory it can count on. You can fine-tune it by observing how much OS-level cache is available during normal server operation, and adding it to `postgresql_shared_buffers`. |
| `postgresql_effective_cache_size` | 50% of RAM |  |
| `postgresql_log_min_duration_statement` | 500 | Log query ran for at least the specified number of milliseconds. |


You can define client authentication with what is traditionally named `pg_hba.conf`. There are two variables to control this: `postgresql_pg_hba_conf_default` and `postgresql_pg_hba_conf`

| Variable                                    | Default value                    | Descriptions                                                  |
| --------------------------------------------|----------------------------------|---------------------------------------------------------------|
| `postgresql_pg_hba_conf_default`            | []                               | A list of allowed clients.keys `type`, `database`, `user`,    |
|                                             |                                  | `address`, and `method`                                       |
| `postgresql_pg_hba_conf`                    | empty list                       | This is defined the same as `postgresql_pg_hba_conf_default`  |
| `postgresql_auth_method_default`            | md5                              | `postgresql_auth_method_default`                              |

Defaults value:

```yml
postgresql_pg_hba_conf_default:
 - { type: local, database: all, user: "{{postgresql_admin_user}}", address: '',             method: peer  }
 - { type: host,  database: all, user: all,                         address: "127.0.0.1/32", method: md5 }
 - { type: host,  database: all, user: all,                         address: "::1/128",      method: md5 }
```

You can define what databases and users to create using `postgresql_databases` and `postgresql_users`.

| Variable                                    | Default value                    | Descriptions                                                  |
| --------------------------------------------|----------------------------------|---------------------------------------------------------------|
| `postgresql_databases`                      | []                               | A list of databases to manage                                 |
| `name`                                      |                                  | name of the database. It mandatory                            |
| `owner`                                     |                                  | owner of the database                                         |
| `encoding`                                  | UTF-8                            | language encoding                                             |
| `lc_collate`                                | en_US.UTF-8                      | Collation order                                               |
| `lc_ctype`                                  | `lc_collate`                     | Character classification, defaults to same as  `lc_collate`   |
| `template`                                  | template0                        | template used to create the database                          |
| `state`                                     | present

Example
-------

```yml
postgresql_databases:
  - name: example
```

| Variable                                    | Default value                    | Descriptions                                                  |
| --------------------------------------------|----------------------------------|---------------------------------------------------------------|
| `postgresql_users`                          | []                               | A list of users to manage                                     |
| `name`                                      |                                  | name of the database user.It mandatory                        |
| `password`                                  |                                  | the user's password. It mandatory                             |
|  `encrypted`                                |                                  | a boolean for whether the password is stored hashed           |
| `priv`                                      |                                  | privileges to grant user on a database                        |
| `db`                                        |                                  | name of database privileges are associated with               |
| `role_attr_flags`                           |                                  | role attributes for the user                                  |
| `state`                                     | present                          |                                                               |

| Variable | Default value | Description |
| -------- | ------------- | ----------- |
| `use_satellite_repo` | false | Use official internet repository to install package of this role. In case host have no internet access and package must be install from a local satellite, set this option to `true`. Repository must be previously declare in satellite and VM must be register to it before deploy this role. |
| `manage_by_cluster` | False | Variable set to true if service state must be managed by a cluster solution (like PCS)  |

Example
-------

```yml
postgresql_users:
  - name: icarus
    password: poor-password
    db: example
    priv: ALL
  - name: another_user
    password: poor-password
    db: anotherdb
    priv: ALL

```

Dependencies
------------

No dependency on other roles.


Example Playbook
----------------

```yml
- hosts: servers
  roles:
    - role: postgresql
      postgresql_data_dir: /srv/postgresql
      postgresql_databases:
        - name: "odoo"
      postgresql_users:
        - name: odoo
          password: "y6uC65m91CMO2"
          db: "odoo"
          priv: ALL
```


License
-------

BSD

Author Information
------------------

* ASSOGBA Boris <borisassogba@live.fr>
