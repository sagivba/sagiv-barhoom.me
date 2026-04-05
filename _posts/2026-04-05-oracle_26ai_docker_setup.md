---
layout: post
title:  "Oracle 26AI Docker Setup - Fast Hands-On Guide for Local AI Database"
author: "Sagiv Barhoom"
date:   2026-04-05
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
- Oracle account (for Container Registry access)
- At least 4GB RAM available

## Step 1 - Login to Oracle Container Registry
Before running the command, make sure you have an Oracle (SSO) account and that you have accepted the repository terms in Oracle Container Registry (OCR). Without this, the docker pull will fail.

Registration and terms acceptance (login and click Accept on database/free repo):
https://container-registry.oracle.com/

Then login using your Oracle SSO credentials:

```bash
docker login container-registry.oracle.com
```

You will be prompted for:
- Username: your Oracle SSO (email)
- Password: your Oracle account password (not the DB password) - I used Google Authenticator token.

## Step 2 - Pull the image
Downloads the Oracle Database image from OCR. T
The first pull may take several minutes.

```bash
docker pull container-registry.oracle.com/database/free:latest
```

## Step 3 - Create a volume
A volume is required to persist database data outside the container. 
Without it, deleting the container may result in data loss, and recreating it will not reuse previous data automatically.

Create a managed volume named `oracle26ai-data`:
```bash
docker volume create oracle26ai-data
```

To verify the volume exists and inspect it:
```bash
docker volume ls
docker volume inspect oracle26ai-data
```
**Important note:** 
Volumes do not have an explicit size limit at creation time. 
They are limited by available disk space or Docker Desktop configuration. 
Ensure sufficient space if you plan to load data or build indexes.

## Step 4 - Run the container
This command:
1. Creates and runs a new container in the background
2. Exposes the DB port
3. Sets an initial password
4. Attaches the volume we created before.

```bash
docker run -d \                                        # Run container in detached mode
  --name oracle26ai \                                  # Assign a friendly container name
  -p 1521:1521 \                                       # Expose Oracle Net port locally
  -e ORACLE_PWD=Oracle123 \                            # Set initial password for admin users
  -v oracle26ai-data:/opt/oracle/oradata \             # Attach volume for persistent data storage
  container-registry.oracle.com/database/free:latest   # Source image
```

Official documentation:
https://docs.oracle.com/en/database/oracle/oracle-database/26/deeck/

Oracle Container Registry:
https://container-registry.oracle.com/

## Step 5 - Check startup
```bash
docker logs -f oracle26ai
docker logs -f oracle26ai | grep 'DATABASE IS READY TO USE!'
```

## Step 6 - Verify container
```bash
docker ps
```

## Connection details
- Host: localhost
- Port: 1521
- Service: FREEPDB1
- User: system
- Password: Oracle123
So with SQLcl you can connect by:
```bash
 sql "system/Oracle123#@//localhost:1521/FREEPDB1"
```

## Validation checks
```sql
select banner_full from v$version;
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
For a small demo environment, a simple approach works well: back up both configuration and data.

This includes:
- Container configuration (JSON)
- Volume data (tar.gz)
- Runtime parameters

Always stop the container before backing up the volume to ensure consistency.

### Full backup script

```bash
#!/bin/bash
# Backup script for the oracle26ai Docker environment.
# It saves container metadata, volume metadata, and the image name,
# then stops the container, archives the Docker volume data,
# and starts the container again.
# Sagiv Barhoom 2026

set -e  # Exit immediately if a command fails.

CONTAINER_NAME=oracle26ai
VOLUME_NAME=oracle26ai-data
BACKUP_ROOT=./backups
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR=${BACKUP_ROOT}/${TIMESTAMP}

mkdir -p "${BACKUP_DIR}"  # Creates the timestamped backup directory if it does not already exist.

# Save container configuration and metadata to a JSON file.
docker inspect "${CONTAINER_NAME}" > "${BACKUP_DIR}/container_inspect.json"

# Save Docker volume configuration and metadata to a JSON file.
docker volume inspect "${VOLUME_NAME}" > "${BACKUP_DIR}/volume_inspect.json"

# Save the image name used by the container to a text file.
docker inspect --format='{{.Config.Image}}' "${CONTAINER_NAME}" > "${BACKUP_DIR}/image_name.txt"

# Stop the container before backing up the volume data.
docker stop "${CONTAINER_NAME}"

if docker run --rm \                   # Runs a temporary container and removes it automatically when finished.
  -v "${VOLUME_NAME}":/volume \        # Mounts the Docker volume into the temporary container at /volume.
  -v "$(pwd)/${BACKUP_DIR}":/backup \  # Mounts the local backup directory into the temporary container at /backup.
  alpine sh -c 'tar czf /backup/oradata.tar.gz -C /volume .'
                                       # Uses Alpine Linux to create a compressed tar.gz archive
                                       # of all data inside /volume and saves it into /backup.
then
  # Print success status and the backup file location.
  echo "Backup completed successfully. Backup file: $(pwd)/${BACKUP_DIR}/oradata.tar.gz"
else
  # Print failure status and the backup directory location.
  echo "Backup failed. Check backup directory: $(pwd)/${BACKUP_DIR}"
  exit 1
fi

# Start the container again after the backup is complete.
docker start "${CONTAINER_NAME}"
```

### Restore script

```bash
#!/bin/bash
# Restore script for the oracle26ai Docker environment.
# It reads the image name from the backup folder, creates the Docker volume,
# restores the archived Oracle data into the volume,
# and starts a new oracle26ai container using the restored data.
# Sagiv Barhoom 2026

set -e  # Exit immediately if a command fails.

BACKUP_DIR=$1
NEW_VOLUME_NAME=oracle26ai-data

# Validate that a backup directory argument was provided.
if [ -z "${BACKUP_DIR}" ]; then
  echo "Restore failed. Usage: $0 <backup_directory>"
  exit 1
fi

# Read the Docker image name from the backup metadata file.
IMAGE_NAME=$(cat "${BACKUP_DIR}/image_name.txt")

# Create the Docker volume that will hold the restored Oracle data.
docker volume create "${NEW_VOLUME_NAME}"

if docker run --rm \                   # Runs a temporary container and removes it automatically when finished.
  -v "${NEW_VOLUME_NAME}":/volume \    # Mounts the target Docker volume into the temporary container at /volume.
  -v "$(pwd)/${BACKUP_DIR}":/backup \  # Mounts the selected backup directory into the temporary container at /backup.
  alpine sh -c 'cd /volume && tar xzf /backup/oradata.tar.gz'
                                       # Uses Alpine Linux to extract the compressed backup archive
                                       # from /backup into the restored Docker volume at /volume.
then
  # Print restore success status and the restored volume name.
  echo "Restore data extraction completed successfully. Restored volume: ${NEW_VOLUME_NAME}"
else
  # Print restore failure status and the backup archive location.
  echo "Restore failed during data extraction. Backup file: $(pwd)/${BACKUP_DIR}/oradata.tar.gz"
  exit 1
fi

# Start a new oracle26ai container using the restored volume and saved image name.
docker run -d \
  --name oracle26ai \                  # Creates and starts a container named oracle26ai.
  -p 1521:1521 \                       # Maps port 1521 from the container to port 1521 on the host.
  -e ORACLE_PWD=Oracle123 \            # Sets the Oracle password inside the container.
  -v "${NEW_VOLUME_NAME}":/opt/oracle/oradata \  # Mounts the restored Docker volume as the Oracle data directory.
  "${IMAGE_NAME}"

# Print final restore status after the container starts.
echo "Restore completed successfully. Container: oracle26ai, Volume: ${NEW_VOLUME_NAME}"
```

### Notes
- Suitable for lab/demo use, not an enterprise backup strategy
- Can be extended using container_inspect.json for full recreation
- Ensure names do not collide during restore

