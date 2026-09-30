# @degreesign/migrate

> One-command Ubuntu/Debian server migration for Apache + PM2 + Node.js — stream web files, SSL certificates, Apache config, PM2 apps and global npm packages from an old server to a new one over SSH.

[![npm version](https://img.shields.io/npm/v/@degreesign/migrate.svg)](https://www.npmjs.com/package/@degreesign/migrate)
[![npm downloads](https://img.shields.io/npm/dm/@degreesign/migrate.svg)](https://www.npmjs.com/package/@degreesign/migrate)
[![license](https://img.shields.io/npm/l/@degreesign/migrate.svg)](LICENSE)
[![platform](https://img.shields.io/badge/platform-Ubuntu%20%7C%20Debian-informational)](https://github.com/DegreeSign/ds_migrate)

## Table of Contents

- [What is @degreesign/migrate?](#what-is-degreesignmigrate)
- [Why use it?](#why-use-it)
- [Requirements](#requirements)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [How it works](#how-it-works)
- [Configuration reference](#configuration-reference)
- [CLI reference](#cli-reference)
- [FAQ](#faq)
- [Keywords](#keywords)
- [License](#license)

## What is @degreesign/migrate?

`@degreesign/migrate` is a zero-config, one-command server migration CLI for **Ubuntu and Debian** hosts running **Apache + PM2 + Node.js**. It connects to your old and new servers over SSH, backs up everything into a single bundle, streams it directly between the hosts, and restores it exactly — including PM2 process state and globally installed npm packages.

No agents, no control panel, no downtime orchestration service. Just passwordless SSH keys and a `.env` file.

## Why use it?

- **One command** — `npx @degreesign/migrate` runs the full 10-stage migration.
- **Exact restoration** — PM2 apps are restored from `dump.pm2`, preserving original paths.
- **Safe by default** — refuses to touch dangerous paths like `/`, `/etc`, `/root` and `/home`.
- **Automatic local backup** — every migration keeps a bundle in `./migration_data/`.
- **Direct streaming** — data travels old server → your machine → new server with rsync progress.
- **Clean replace** — target directories are synced with `--delete` so the new server matches the old one exactly.
- **Global packages reinstalled** — global npm packages from the old server are captured and reinstalled.
- **Real-time progress** — a 10-stage progress bar shows exactly where the migration is.

## Requirements

- **Source and target servers:** Ubuntu or Debian.
- **SSH access** to both servers with **passwordless (key-based) authentication**.
- **sudo** privileges on the new server for installing packages.
- **Node.js** on the machine you run the CLI from (for `npx` / npm install).
- **rsync** available locally and on both servers.

## Installation

Install globally:

```bash
# npm
npm install -g @degreesign/migrate

# yarn
yarn global add @degreesign/migrate

# pnpm
pnpm add -g @degreesign/migrate
```

Or run it without installing:

```bash
npx @degreesign/migrate
```

> `@degreesign/migrate` is a CLI/server tool and has **no browser build**, so there is no CDN snippet.

## Quick Start

The CLI reads a `.env` file from the **current working directory**, so run it from a dedicated folder.

```bash
# 1. Create a migration folder
mkdir -p tmp_migration && cd tmp_migration

# 2. Create .env (copy the example if installed locally)
cp ../node_modules/@degreesign/migrate/.env.example .env

# 3. Edit .env with your SSH details and paths
#    OLD_USER / OLD_IP / NEW_USER / NEW_IP / MIGRATE_DIRS / MIGRATE_FILES

# 4. Run the migration
npx @degreesign/migrate
```

Example `.env`:

```env
# Node.js
NODE_VERSION=24

# SSH details
OLD_USER='old_username'
OLD_IP='0.0.0.0'
NEW_USER='new_username'
NEW_IP='1.1.1.1'

# Comma-separated – no spaces after commas
MIGRATE_DIRS='/root/pm2_files,/var/www,/etc/letsencrypt'
MIGRATE_FILES='/etc/apache2/apache2.conf,/etc/ssh/sshd_config'
```

> The values above are **placeholders** — review and update them before running.

## How it works

The migration runs in **10 automatic stages**:

| # | Stage | What happens |
| - | ----- | ------------ |
| 1 | Migration Start | Loads and validates `.env`, parses/validates paths, resolves login targets. |
| 2 | Data Backup | rsyncs directories, copies files, saves PM2 state and captures global npm packages on the old server. |
| 3 | Data Compression | Tars everything into `/tmp/migration-bundle.tar.gz` on the old server. |
| 4 | Data Download | Downloads the bundle to `./migration_data/` locally (with one retry). |
| 5 | Data Upload | Uploads the bundle to the new server (with one retry). |
| 6 | Softwares Installation | Installs Apache, jq, curl, NVM and the configured Node.js version. |
| 7 | Restore Data | Extracts and restores directories/files into their exact original paths. |
| 8 | NPM Installation | Reinstalls PM2 and the captured global/root npm packages. |
| 9 | Restore Processes | Restarts Apache and runs `pm2 resurrect` + `pm2 save`. |
| 10 | Finalising Migration | Cleans up temp files and prints a timed summary. |

## Configuration reference

All configuration lives in a `.env` file in the current working directory.

| Variable | Required | Description |
| -------- | -------- | ----------- |
| `OLD_USER` | Yes | SSH username on the old (source) server. |
| `OLD_IP` | Yes | Hostname or IP of the old server. |
| `NEW_USER` | Yes | SSH username on the new (target) server. |
| `NEW_IP` | Yes | Hostname or IP of the new server. |
| `MIGRATE_DIRS` | Yes | Comma-separated absolute directories to migrate, e.g. `/var/www`. No spaces after commas. |
| `MIGRATE_FILES` | Yes | Comma-separated absolute files to migrate, e.g. `/etc/apache2/apache2.conf`. No spaces after commas. |
| `NODE_VERSION` | No | Node.js version to install via NVM on the new server. Defaults to `24`. |

**Safety guardrails:** paths must be absolute, and the script blocks `/`, `/home`, `/etc` and `/root` as migration targets.

## CLI reference

| Command | Description |
| ------- | ----------- |
| `npx @degreesign/migrate` | Run the migration from the current directory's `.env`. |
| `server-migration` | Globally installed binary — same as above. |
| `npm run migrate` | Local script that makes `migrate.sh` executable and runs it. |

## FAQ

**What does @degreesign/migrate do?**
It migrates an Ubuntu/Debian Apache + PM2 + Node.js stack from one server to another in a single command, moving files, config, SSL certs, PM2 processes and global npm packages.

**Is it free?**
Yes. It is open source under the Apache-2.0 license.

**Does it work in the browser?**
No. It is a Node.js/server CLI that requires SSH and rsync access to your servers.

**Does it have dependencies?**
The package itself ships as a shell script and has no runtime npm dependencies. It relies on standard tools on your servers: `ssh`, `rsync`, `tar`, `jq`, `curl` and `pm2`. Apache and Node.js are installed on the target during migration.

**Is it written in TypeScript?**
No. It is a Bash script (`migrate.sh`) published as an npm package.

**Which frameworks does it support?**
It is framework-agnostic — it migrates the server stack (Apache, PM2, Node.js), so any Node.js app managed by PM2 is supported.

## Keywords

server migration, migrate server, apache migration, pm2 migration, nodejs, nvm, ubuntu, debian, ssh, rsync, letsencrypt, ssl certificates, devops, cli

## License

Apache-2.0 © [DegreeSign](https://github.com/DegreeSign) — see [LICENSE](LICENSE).
