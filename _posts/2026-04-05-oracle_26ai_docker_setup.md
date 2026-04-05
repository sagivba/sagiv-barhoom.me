---
layout: post
title:  "Oracle 26AI Docker Setup - Fast Hands-On Guide for Local AI Database"
author: "Sagiv Barhoom"
date:   2026-04-05
categories: ORACLE
background: '/img/posts/oracle26ai-docker.jpg'
---

# Oracle 26AI Docker Setup - Fast Hands-On Guide for Local AI Database (Select AI Ready)

> Note: All commands and scripts in this post are based on working with WSL (Windows Subsystem for Linux), and are therefore written in Bash.

## Goal of this post

The goal is to quickly set up an Oracle AI Database 26ai environment on a local machine using Docker, and reach a point where you can connect to the database and start working.

The focus is on:
- Fast setup
- Minimal complexity
- A sandbox environment for learning

---

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

---

## Prerequisites

- WSL (Windows Subsystem for Linux) installed and configured
- Docker installed
- Oracle account (for Container Registry access)
- At least 4GB RAM available

---

## Step 1 - Login to Oracle Container Registry

Before running the command, make sure you have an Oracle (SSO) account and that you have accepted the repository terms in Oracle Container Registry (OCR). Without this, the pull will fail.

Registration and terms acceptance (login and click Accept on database/free repo):
https://container-registry.oracle.com/

Then login using your Oracle SSO credentials:

```bash
docker login container-registry.oracle.com
```

You will be prompted for:
- Username: your Oracle SSO (email)
- Password: your Oracle account password (not the DB password)

---

## Step 2 - Pull the image

This command downloads the Oracle Database image from Oracle Container Registry to your local machine. The first pull may take several minutes depending on network speed.

```bash
docker pull container-registry.oracle.com/database/free:latest
```

---

## Step 3 - Create a volume

A volume is required to persist database data outside the container. Without it, deleting the container may result in data loss, and recreating it will not reuse previous data automatically.

Create a managed volume named `oracle26ai-data`:

```bash
docker volume create oracle26ai-data
```

To verify the volume exists:

```bash
docker volume ls
```

Or inspect it:

```bash
docker volume inspect oracle26ai-data
```

Important note: volumes do not have an explicit size limit at creation time. They are limited by available disk space or Docker Desktop configuration. Ensure sufficient space if you plan to load data or build indexes.

---

## Step 4 - Run the container

This command creates and runs a new container in the background, exposes the DB port, sets an initial password, and attaches the volume.

```bash
docker run -d \                                # Run container in detached mode
  --name oracle26ai \                         # Assign a friendly container name
  -p 1521:1521 \                              # Expose Oracle Net port locally
  -e ORACLE_PWD=Oracle123 \                   # Set initial password for admin users
  -v oracle26ai-data:/opt/oracle/oradata \    # Attach volume for persistent data storage
  container-registry.oracle.com/database/free:latest   # Source image
```

Official documentation:
https://docs.oracle.com/en/database/oracle/oracle-database/26/deeck/

Oracle Container Registry:
https://container-registry.oracle.com/

---

## Step 5 - Check startup

```bash
docker logs -f oracle26ai
```

Look for:

```text
DATABASE IS READY TO USE!
```

---

## Step 6 - Verify container

```bash
docker ps
```

---

## Connection details

- Host: localhost
- Port: 1521
- Service: FREEPDB1
- User: system
- Password: Oracle123

---

## Validation checks

```sql
select banner_full from v$version;
```

Expected output:

```text
Oracle AI Database 26ai Free Release 23.26.1.0.0
```

```sql
show pdbs;
```

Expected:
- FREEPDB1 in READ WRITE mode

---

## Basic service management

Once created, the container does not need to be recreated every time.

Stop the container:

```bash
docker stop oracle26ai    # Stops container, data remains in volume
```

Start it again:

```bash
docker start oracle26ai   # Starts existing container with same data
```

Check running containers:

```bash
docker ps
```

List all containers:

```bash
docker ps -a
```

---

## Full backup and restore

For a small demo environment, a simple approach works well: back up both configuration and data.

This includes:
- Container configuration (JSON)
- Volume data (tar.gz)
- Runtime parameters

Always stop the container before backing up the volume to ensure consistency.

### Full backup script

```bash
#!/bin/bash
set -e

CONTAINER_NAME=oracle26ai
VOLUME_NAME=oracle26ai-data
BACKUP_ROOT=./backups
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR=${BACKUP_ROOT}/${TIMESTAMP}

mkdir -p "${BACKUP_DIR}"

docker inspect "${CONTAINER_NAME}" > "${BACKUP_DIR}/container_inspect.json"
docker volume inspect "${VOLUME_NAME}" > "${BACKUP_DIR}/volume_inspect.json"
docker inspect --format='{{.Config.Image}}' "${CONTAINER_NAME}" > "${BACKUP_DIR}/image_name.txt"

docker stop "${CONTAINER_NAME}"

docker run --rm \
  -v "${VOLUME_NAME}":/volume \
  -v "$(pwd)/${BACKUP_DIR}":/backup \
  alpine sh -c 'tar czf /backup/oradata.tar.gz -C /volume .'

docker start "${CONTAINER_NAME}"
```

### Restore script

```bash
#!/bin/bash
set -e

BACKUP_DIR=$1
NEW_VOLUME_NAME=oracle26ai-data
IMAGE_NAME=$(cat "${BACKUP_DIR}/image_name.txt")

docker volume create "${NEW_VOLUME_NAME}"

docker run --rm \
  -v "${NEW_VOLUME_NAME}":/volume \
  -v "$(pwd)/${BACKUP_DIR}":/backup \
  alpine sh -c 'cd /volume && tar xzf /backup/oradata.tar.gz'

docker run -d \
  --name oracle26ai \
  -p 1521:1521 \
  -e ORACLE_PWD=Oracle123 \
  -v "${NEW_VOLUME_NAME}":/opt/oracle/oradata \
  "${IMAGE_NAME}"
```

### Notes

- Suitable for lab/demo use, not enterprise backup strategy
- Can be extended using container_inspect.json for full recreation
- Ensure names do not collide during restore

---

## Summary

Within minutes, you can have a fully working Oracle 26AI environment on your laptop.

This setup enables:
- Immediate SQL work
- AI experimentation
- Fast iteration without complex setup

---

## Next steps

- Build a budget schema
- Configure Select AI with OpenAI
- Test natural language to SQL queries
