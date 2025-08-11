### backup_mssql_docker

This role performs full backups of Microsoft SQL Server databases running inside Docker containers. The backup is executed with `sqlcmd` inside the target container via the `community.docker.docker_container_exec` module, and the resulting `.bak` file is written under `/var/opt/mssql/backups` inside the container.

### How it works

- Ensures the backup directory `/var/opt/mssql/backups` exists inside the container.
- Runs the `BACKUP DATABASE` T‑SQL using `sqlcmd` inside the container. The T‑SQL is rendered from `templates/backup_query.j2`. When `compression: true`, the query adds `COMPRESSION`. When `certificate_name` is defined, the query adds `ENCRYPTION` with the specified server certificate.
- Success is determined from `sqlcmd` output; the task checks for the substring `backup database successfully` and marks the task as changed on success.

### Requirements

- Ansible >= 2.9 (see `meta/main.yml`).
- Access to the Docker host that runs the SQL Server container.
- Installed Ansible collection for Docker:
  - `ansible-galaxy collection install community.docker`
- `sqlcmd` present inside the target container (package `mssql-tools18`), typically at `/opt/mssql-tools18/bin/sqlcmd` per Microsoft guidance, or available on `PATH`.
- Valid SQL Server credentials and the database name to back up.

### Role variables

Defaults are defined in `defaults/main.yml`.

| Variable               | Default         | Required | Description |
|------------------------|-----------------|----------|-------------|
| `mssql_container_name` | `mssql`         | Yes      | Name of the container running SQL Server. |
| `mssql_username`       | `sa`            | Yes      | SQL Server login. |
| `mssql_password`       | `''`            | Yes      | SQL Server password. Store with Ansible Vault. |
| `database_name`        | `master`        | Yes      | Database to back up. |
| `compression`          | `true`          | No       | Adds `COMPRESSION` to `BACKUP DATABASE`. |
| `medianame`            | `MSSQLBackups`  | No       | Sets `MEDIANAME` in the `BACKUP` command. |
| `mssql_hide_sensitive` | `true`          | No       | When `true`, uses `no_log` to prevent leaking secrets in logs. |
| `certificate_name`     | —               | No       | If set, enables `ENCRYPTION` with the specified server certificate. See Backup Encryption docs. |
| `date_string`          | —               | No       | Optional timestamp string used in the backup filename; otherwise a Jinja2 `now('%s')` timestamp is used in the template. |

### Output

- Backup file is written inside the container with a name similar to:
  - `/var/opt/mssql/backups/backup_mssql_<compressed|uncompressed>_<database>_<timestamp>.bak`

### Usage

Minimal example play:

```yaml
- name: Backup SQL Server database inside a Docker container
  hosts: db_hosts
  gather_facts: false
  collections:
    - community.docker
  vars:
    mssql_container_name: mssql
    mssql_username: sa
    mssql_password: "{{ vault_mssql_password }}"  # store with Ansible Vault
    database_name: mydb
    compression: true
    # certificate_name: MyServerCert  # enable if you configured backup encryption
  roles:
    - role: backup_mssql_docker
```

Quick test using the included scenario:

```bash
ansible-playbook roles/backup_mssql_docker/tests/test.yml -i roles/backup_mssql_docker/tests/inventory
```

### Security and operations

- Manage secrets with Ansible Vault (official): https://docs.ansible.com/ansible/latest/vault_guide/index.html
- `no_log` is enabled by default via `mssql_hide_sensitive: true`. Temporarily set to `false` only for troubleshooting, then revert. See `no_log` docs linked above.
- To use backup encryption, provision and reference a valid SQL Server certificate and review Microsoft’s Backup Encryption documentation.
- Permissions: per Microsoft, `BACKUP DATABASE`/`BACKUP LOG` permissions are held by `sysadmin` (server), and `db_owner`/`db_backupoperator` (database). See the Permissions section in the BACKUP T‑SQL documentation.

### Limitations

- Backups are inherently non‑idempotent; each run produces a new `.bak` file and the task is reported as changed on success.
- The role expects `sqlcmd` to be available inside the container (commonly at `/opt/mssql-tools18/bin/sqlcmd`). If installed elsewhere, ensure it is present on `PATH` and adjust the role if needed.

### References (official)

- Ansible community.docker `docker_container_exec` module: https://docs.ansible.com/ansible/latest/collections/community/docker/docker_container_exec_module.html
- Ansible collection `community.docker`: https://docs.ansible.com/ansible/latest/collections/community/docker/index.html
- Ansible builtin `template` lookup: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_lookup.html
- Ansible playbook keyword `no_log`: https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_keywords.html
- Microsoft Docs — BACKUP (Transact‑SQL): https://learn.microsoft.com/sql/t-sql/statements/backup-transact-sql
- Microsoft Docs — Backup Encryption: https://learn.microsoft.com/sql/relational-databases/backup-restore/backup-encryption
- Microsoft Docs — Backup Compression: https://learn.microsoft.com/sql/relational-databases/backup-restore/backup-compression-sql-server
- Microsoft Docs — Install SQL Server command‑line tools (sqlcmd) on Linux: https://learn.microsoft.com/sql/linux/sql-server-linux-setup-tools
- Microsoft Docs — File locations for SQL Server on Linux (`/var/opt/mssql`): https://learn.microsoft.com/sql/linux/sql-server-linux-file-locations
- Ansible date/time filters (`now()`): https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_filters.html#dates-and-times

### License

GPL-2.0-or-later (as declared in `meta/main.yml`).


