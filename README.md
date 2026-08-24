<!--
SPDX-FileCopyrightText: 2023 Slavi Pantaleev
SPDX-FileCopyrightText: 2026 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Lago Ansible role

This is an [Ansible](https://www.ansible.com/) role which installs [Lago](https://getlago.com/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

This role *implicitly* depends on:

- [`com.devture.ansible.role.playbook_help`](https://github.com/devture/com.devture.ansible.role.playbook_help)
- [`com.devture.ansible.role.systemd_docker_base`](https://github.com/devture/com.devture.ansible.role.systemd_docker_base)

Check [`defaults/main.yml`](defaults/main.yml) for the full list of supported options.

💡 For an Ansible playbook which integrates this role and makes it easier to use, see the [Mother-of-All-Self-Hosting Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

## Requirements

### A Postgres server which has `pg_partman`

Lago's database schema (`db/structure.sql` in [`getlago/lago-api`](https://github.com/getlago/lago-api)) runs `CREATE EXTENSION pg_partman`, and has done so since Lago v1.41.0. `pg_partman` is not part of a stock Postgres installation, so the Postgres server this role is pointed at must have the extension available. Upstream publishes [`getlago/postgres-partman`](https://hub.docker.com/r/getlago/postgres-partman) for this purpose and uses it in its own `docker-compose.yml`.

⚠️ Without it, the failure is silent rather than loud. The API container's entrypoint runs `rails db:migrate` and then starts the web server whether or not the migration succeeded, and Lago's `/health` endpoint only issues an empty statement on its database connection — so it answers `200 Success` against a database with no tables in it at all. The result is a Lago which looks up and is entirely unusable.

The Molecule scenario in this repository pins Postgres to upstream's image for exactly this reason; see [`molecule/default/molecule.yml`](molecule/default/molecule.yml).

## Development

### pre-commit

You can optionally install a Git pre-commit hook (via [mise](https://mise.jdx.dev/) + [prek](https://prek.j178.dev/)) that runs formatting and linting checks before each commit. See [`.pre-commit-config.yaml`](./.pre-commit-config.yaml) for which hooks are to be executed.

To install the hook, run the [`just`](https://github.com/casey/just) command below:

```sh
just prek-install-git-pre-commit-hook
```

### Molecule

This role supports [Molecule](https://docs.ansible.com/projects/molecule/), an Ansible testing framework designed for developing and testing Ansible collections, playbooks, and roles.

Refer to [this page](./molecule/README.md) for details about how to utilize it.
