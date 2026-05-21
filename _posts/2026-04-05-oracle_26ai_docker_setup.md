---
layout: post
title: "Oracle 26AI Docker Setup - Fast Hands-On Guide for Local AI Database"
author: "Sagiv Barhoom"
date: 2026-04-05
categories: ORACLE
background: '/img/posts/oracle26ai-docker.png'
---

# Oracle 26AI Docker Setup - Fast Hands-On Guide for Local AI Database (Select AI Ready)

> Note: All commands and scripts in this post are based on working with WSL, and are therefore written in Bash.

## Goal of this post

The goal is to quickly set up an Oracle AI Database 26ai environment on a local machine using Docker, and reach a point where we can connect to the database and start working.

The focus is on:

- Fast setup
- Minimal complexity
- A sandbox environment for learning

## Architecture choice - available options

There are three main approaches:

- Direct installation on Windows 64-bit
- VM setup (VirtualBox + Oracle Linux)
- Docker with a prebuilt image

For fast hands-on work, Docker was chosen for the following reasons:

- Significantly faster setup
- No need to install a full operating system
- Database is ready to use within minutes
- Ideal for POC and learning scenarios

## Prerequisites

- WSL (Windows Subsystem for Linux) installed and configured
- Docker installed
- Oracle account, for Container Registry access
- At least 4GB RAM available

## Step 1 - Login to Oracle Container Registry

Before running the command, make sure you have an Oracle SSO account and that you have accepted the repository terms in Oracle Container Registry.

Without this, the `docker pull` command will fail.

Registration and terms acceptance:

https://container-registry.oracle.com/

Login to Oracle Container Registry, open the `database/free` repository, and accept the terms.

Then login from the terminal using your Oracle SSO credentials:

```bash
docker login container-registry.oracle.com
```

You will be prompted for:

- Username: your Oracle SSO email
- Password: your Oracle account password, not the database password

In my case, the login also required a Google Authenticator token.

## Step 2 - Pull the image

Download the Oracle Database image from Oracle Container Registry:

```bash
docker pull container-registry.oracle.com/database/free:latest
```

The first pull may take several minutes.

For a quick lab setup, `latest` is convenient. For reproducible project work, prefer an explicit image tag once you decide which version you want to standardize on.

You can inspect the local image after pulling it:

```bash
docker image inspect container-registry.oracle.com/database/free:latest \
  --format '{{.RepoTags}} {{.Id}} {{.Created}}'
```

## Step 3 - Create a volume

A volume is required to persist database data outside the container.

Without it, deleting the container may result in data loss, and recreating it will not reuse previous data automatically.

Create a managed volume named `oracle26ai-data`:

```bash
docker volume create oracle26ai-data
```

To verify that the volume exists and inspect it:

```bash
docker volume ls
docker volume inspect oracle26ai-data
```

**Important note:** Docker volumes do not have an explicit size limit at creation time. They are limited by available disk space or by Docker Desktop configuration. Ensure sufficient space if you plan to load data or build indexes.

## Step 4 - Run the container

Set a local password before running the container.

Do not reuse the example password in shared scripts, repositories, or real environments.

```bash
DB_PASSWORD='ChangeMe_26ai_123'
```

This command:

1. Creates and runs a new container in the background
2. Exposes the database port locally
3. Sets the initial password for the Oracle administrative users
4. Attaches the volume created earlier

```bash
docker run -d \
  --name oracle26ai \
  -p 1521:1521 \
  -e ORACLE_PWD="${DB_PASSWORD}" \
  -v oracle26ai-data:/opt/oracle/oradata \
  container-registry.oracle.com/database/free:latest
```

Official documentation:

https://docs.oracle.com/en/database/oracle/oracle-database/26/deeck/

Oracle Container Registry:

https://container-registry.oracle.com/

## Step 5 - Check startup

Follow the container logs:

```bash
docker logs -f oracle26ai
```

Wait until the database is ready:

```bash
docker logs -f oracle26ai | grep 'DATABASE IS READY TO USE!'
```

## Step 6 - Verify container

```bash
docker ps
```

## Connection details

- Host: `localhost`
- Port: `1521`
- Service: `FREEPDB1`
- User: `system`
- Password: the value you used in `DB_PASSWORD`

With SQLcl, connect using:

```bash
sql "system/${DB_PASSWORD}@//localhost:1521/FREEPDB1"
```

If you open a new shell and the `DB_PASSWORD` variable is no longer defined, use the password you set when creating the container.

## Validation checks

```sql
select banner_full
from v$version;

-- Expected output:
-- Oracle AI Database 26ai Free Release 23.26.1.0.0
```

```sql
show pdbs;

-- Expected output:
-- FREEPDB1 in READ WRITE mode
```

## Basic service management

Once created, the container does not need to be recreated every time.

| Command | What it does |
|---|---|
| `docker start oracle26ai` | Starts the existing `oracle26ai` container with the same persisted data. |
| `docker stop oracle26ai` | Stops the `oracle26ai` container. The data remains stored and is not removed. |
| `docker ps` | Shows only the containers that are currently running. |
| `docker ps -a` | Shows all containers, including running and stopped ones. |
| `docker logs oracle26ai` | Displays the logs of the `oracle26ai` container. |
| `docker logs -f oracle26ai` | Displays the container logs in real time and keeps following new log output. |

---

## Full backup and restore

For a small demo environment, a simple approach can be enough: back up both configuration and data.

This includes:

- Container configuration JSON
- Volume metadata JSON
- Image name
- Volume data as a `tar.gz` archive

This backup approach is suitable for lab/demo environments only.

It is not an enterprise backup strategy and does not replace RMAN, Data Pump, archive log based recovery, or any tested production-grade backup process.

Always stop the container before backing up the volume to avoid copying database files while they are being changed.

### Full backup script

```bash
#!/bin/bash

# Backup script for the oracle26ai Docker environment.
# It saves container metadata, volume metadata, and the image name,
# then stops the container, archives the Docker volume data,
# and starts the container again.
#
# Sagiv Barhoom 2026

set -e

CONTAINER_NAME=oracle26ai
VOLUME_NAME=oracle26ai-data
BACKUP_ROOT=./backups
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR=${BACKUP_ROOT}/${TIMESTAMP}

mkdir -p "${BACKUP_DIR}"

BACKUP_DIR_ABS=$(cd "${BACKUP_DIR}" && pwd)

docker inspect "${CONTAINER_NAME}" > "${BACKUP_DIR_ABS}/container_inspect.json"
docker volume inspect "${VOLUME_NAME}" > "${BACKUP_DIR_ABS}/volume_inspect.json"
docker inspect --format='{{.Config.Image}}' "${CONTAINER_NAME}" > "${BACKUP_DIR_ABS}/image_name.txt"

docker stop "${CONTAINER_NAME}"

restart_container() {
  docker start "${CONTAINER_NAME}" >/dev/null
}

trap restart_container EXIT

docker run --rm \
  -v "${VOLUME_NAME}":/volume \
  -v "${BACKUP_DIR_ABS}":/backup \
  alpine sh -c 'tar czf /backup/oradata.tar.gz -C /volume .'

trap - EXIT

docker start "${CONTAINER_NAME}" >/dev/null

echo "Backup completed successfully."
echo "Backup directory: ${BACKUP_DIR_ABS}"
echo "Backup file: ${BACKUP_DIR_ABS}/oradata.tar.gz"
```

### Restore script

```bash
#!/bin/bash

# Restore script for the oracle26ai Docker environment.
# It reads the image name from the backup folder, creates a Docker volume,
# restores the archived Oracle data into the volume,
# and starts a new oracle26ai container using the restored data.
#
# Sagiv Barhoom 2026

set -e

BACKUP_DIR_INPUT=$1
CONTAINER_NAME=oracle26ai
NEW_VOLUME_NAME=oracle26ai-data
HOST_PORT=1521

if [ -z "${BACKUP_DIR_INPUT}" ]; then
  echo "Restore failed. Usage: $0 <backup-directory>"
  exit 1
fi

BACKUP_DIR=$(cd "${BACKUP_DIR_INPUT}" && pwd)
IMAGE_NAME=$(cat "${BACKUP_DIR}/image_name.txt")

if docker ps -a --format '{{.Names}}' | grep -qx "${CONTAINER_NAME}"; then
  echo "Restore failed. A container named ${CONTAINER_NAME} already exists."
  echo "Remove or rename the existing container before restoring."
  exit 1
fi

if docker volume inspect "${NEW_VOLUME_NAME}" >/dev/null 2>&1; then
  echo "Restore failed. A volume named ${NEW_VOLUME_NAME} already exists."
  echo "Remove or rename the existing volume before restoring."
  exit 1
fi

docker volume create "${NEW_VOLUME_NAME}"

docker run --rm \
  -v "${NEW_VOLUME_NAME}":/volume \
  -v "${BACKUP_DIR}":/backup \
  alpine sh -c 'cd /volume && tar xzf /backup/oradata.tar.gz'

docker run -d \
  --name "${CONTAINER_NAME}" \
  -p "${HOST_PORT}:1521" \
  -v "${NEW_VOLUME_NAME}":/opt/oracle/oradata \
  "${IMAGE_NAME}"

echo "Restore completed successfully."
echo "Container: ${CONTAINER_NAME}"
echo "Volume: ${NEW_VOLUME_NAME}"
```

Restoring the volume restores the database files as they were at backup time.

It does not reset the database password. Use the same database credentials that were valid when the backup was created.

### Notes

- Suitable for lab/demo use, not an enterprise backup strategy
- Can be extended using `container_inspect.json` for full recreation
- Ensure container and volume names do not collide during restore
- Validate the restored database before relying on it for additional work
