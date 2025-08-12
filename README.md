### Ansible Collection: `mahdishariatzade.shared_tasks`

This repository provides the Ansible collection `mahdishariatzade.shared_tasks` containing reusable roles for common automation tasks.

Links:
- Galaxy page: [mahdishariatzade/shared_tasks](https://galaxy.ansible.com/ui/repo/published/mahdishariatzade/shared_tasks/)
- Official docs — Installing collections: [collections_installing.html](https://docs.ansible.com/ansible/latest/collections_guide/collections_installing.html)
- Official docs — Using collections: [collections_using.html](https://docs.ansible.com/ansible/latest/collections_guide/collections_using.html)
- Official docs — `ansible-galaxy` CLI: [ansible-galaxy](https://docs.ansible.com/ansible/latest/cli/ansible-galaxy.html)

### Requirements

- Ansible `>= 2.9.10` (as declared in `meta/runtime.yml`).
- Collection dependency: `community.docker (>=3,<4)` (declared in `galaxy.yml`).

### Installation

- Direct install from Ansible Galaxy:

```bash
ansible-galaxy collection install mahdishariatzade.shared_tasks
```

- Install via `requirements.yml`:

Create `requirements.yml`:

```yaml
collections:
  - name: mahdishariatzade.shared_tasks
    version: "0.0.2"
```

Then run:

```bash
ansible-galaxy collection install -r requirements.yml
```

- Install from a local tarball or directory:

```bash
ansible-galaxy collection install ./mahdishariatzade-shared_tasks-0.0.2.tar.gz
# or
ansible-galaxy collection install /path/to/collection_directory
```

Refer to the official documentation for additional installation options and flags: [collections_installing.html](https://docs.ansible.com/ansible/latest/collections_guide/collections_installing.html), [ansible-galaxy](https://docs.ansible.com/ansible/latest/cli/ansible-galaxy.html).

### Usage (roles)

Use the fully qualified role name (FQCN) to reference roles in your playbooks. See: [Using collections](https://docs.ansible.com/ansible/latest/collections_guide/collections_using.html).

```yaml
---
- name: Example using a role from the collection
  hosts: all
  gather_facts: false
  roles:
    - role: mahdishariatzade.shared_tasks.upload_to_s3
```

Alternatively, you can declare the collection at the play level (for tasks, modules, filters, etc.). See: [Using collections](https://docs.ansible.com/ansible/latest/collections_guide/collections_using.html).

```yaml
---
- name: Example declaring a collection
  hosts: all
  collections:
    - mahdishariatzade.shared_tasks
  tasks:
    - name: Example task stub
      debug:
        msg: "Collection declared for this play"
```

### Contents

Roles included in this collection:
- `backup_mssql_docker` — see `roles/backup_mssql_docker/README.md`
- `discover_containers` — see `roles/discover_containers/README.md`
- `transfer_files` — see `roles/transfer_files/README.md`
- `upload_to_s3` — see `roles/upload_to_s3/README.md`

For available default variables and usage details, review each role's `defaults/main.yml` and `README.md`.

### Building from source (optional)

If you are developing locally and want to build a distributable artifact, use the `ansible-galaxy` build command. See: [ansible-galaxy](https://docs.ansible.com/ansible/latest/cli/ansible-galaxy.html).

```bash
ansible-galaxy collection build
# Produces a tar.gz in the current directory
```

You can then install the built artifact locally as shown in the installation section.

### Dependencies

This collection declares the following dependency in `galaxy.yml`:

```yaml
dependencies:
  community.docker: ">=3,<4"
```

Dependencies are resolved automatically by `ansible-galaxy collection install`. See: [collections_installing.html](https://docs.ansible.com/ansible/latest/collections_guide/collections_installing.html).

### License

GPL-2.0-or-later (see `galaxy.yml`).

