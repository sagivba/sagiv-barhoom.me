---
layout: post
title:  "Running the CSM Software with Docker"
author: "Sagiv Barhoom & Elinor Abitbul"
date:   2025-07-30
categories: Dockers 
background: '/img/posts/be-linux.jpg.jpg'
---


# Running the CSM Software with Docker
## General Background
For full access to the source code and documentation, visit the GitHub repository: 
https://github.com/continuous-symmetry-measure/csm

The CSM (Continuous Symmetry Measure) software is a computational chemistry tool for analyzing the symmetry and chirality of molecular structures based on defined point groups (e.g., C2, C3, Td).
This guide explains how to use CSM without traditional software installation, by utilizing a tool called Docker.

## What is Docker?
Docker is a platform that allows software to run inside isolated "containers", self-contained environments that include all the dependencies and files the software needs. Instead of installing CSM manually, you can use a prebuilt Docker container that comes ready to run.
Advantages:
- Avoid conflicts with other software.
- Ensure consistency and reproducibility.
- Easily share the same setup with colleagues.
Disadvantages:
- The computing power available via Docker on a personal machine may be limited, especially for large-scale or parallel analyses.
- The CSM development team typically uses High-Performance Computing (HPC) environments to run many instances of CSM simultaneously, which is not feasible with standard Docker usage on personal hardware.

## Prerequisites
- Operating system: Linux, macOS, or Windows
- Docker is installed on your machine:
  Download here: https://docs.docker.com/get-docker/

## How It Works with Docker
Using Docker involves three key concepts:
1. Image – a preconfigured template with everything needed to run the CSM software.
2. Container – an isolated runtime environment created from the image.
3. Execution – running the container with specific commands to analyze molecules.

### The workflow is as follows (ellaboration of this steps below):
- You pull (download) the CSM image from Docker Hub.
- Docker then creates a container based on that image.
- You run the container, passing in molecule files and parameters.
This setup ensures consistency across different machines.
To verify each step is working:
- Check Docker installation: `docker --version`
- Verify Docker is running properly: `docker info`
- List downloaded images: `docker images`
Test basic container execution:
```
docker run --rm teamcsm/csm:v1.3.7b1 csm --help
```

## Step 1: Download the CSM Docker Container
### Note on Working Behind a Proxy
If you're working in an environment with a network proxy (e.g., a university or institutional firewall), Docker may need to be configured to access external resources.
On Linux/macOS, create or edit the Docker systemd configuration:
```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo nano /etc/systemd/system/docker.service.d/http-proxy.conf
```
Then add:
```ini
[Service]
  Environment="HTTP_PROXY=http://proxy.example.com:8080/"
  Environment="HTTPS_PROXY=http://proxy.example.com:8080/"
```
After saving, reload and restart Docker:
```bash
sudo systemctl daemon-reexec
sudo systemctl restart docker
```
On Windows or macOS with Docker Desktop, go to:
<strong>Settings > Resources > Proxies </strong>and set your proxy configuration there.
Ensure that you replace `proxy.example.com:8080` with your actual proxy address.
Run the following command in your terminal:
```bash
docker pull teamcsm/csm:v1.3.7b1
```
This will download version v1.3.7b1 of the CSM software to your machine.
To verify the image was successfully downloaded, run:
```bash
docker images | grep teamcsm/csm
```
You should see a line indicating that the image teamcsm/csm with the v1.3.7b1 tag is available locally.

## Step 2: Run the Software
To analyze a specific molecule (for example, in the C2 point group), use this command:
```bash
docker run -it teamcsm/csm:v1.3.7b1 csm exact C2 --input /data/molecule.sdf --output /data/results
```
Explanation:
- `exact` — Use the exact symmetry analysis mode.
- `C2` — The target point group.
- `--input` — Input molecule file (formats: SDF, XYZ, MOL, or PDB).
- `--output` — Output directory where results will be saved.
  
## Step 3: Access Local Files (Recommended)
To allow the software to read/write files from your computer, it's recommended to map a local folder into the container. Example:
```bash
docker run -it -v $(pwd):/data teamcsm/csm:v1.3.7b1 csm exact C2 --input /data/molecule.sdf --output /data/results
```
This command maps your current directory to /data inside the container.

## Available Operation Modes
- `exact` - Accurate mode for small/medium-sized molecules.
- `approx` -  Approximate mode for large molecules or homomeric proteins.
- `trivial` - Basic unit-permutation check.
To explore more options, run inside the container:
```bash
csm --help
```
## Useful Links
- GitHub Repository: https://github.com/continuous-symmetry-measure/csm
- Docker Hub Page: https://hub.docker.com/r/teamcsm/csm
- Online CSM Tool: https://csm.ouproj.org.il

## Questions and Support
- Submit issues via the GitHub Issues page.
- Join the Google Group for discussions: https://groups.google.com/g/csm-openu

## Citations
For academic use, please cite the relevant CSM publications as listed in the GitHub README.
