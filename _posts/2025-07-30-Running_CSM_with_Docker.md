---
layout: post
title:  "Running the CSM Software with Docker"
author: "Sagiv Barhoom & Elinor Abitbul, The Open University of Israel"
date:   2025-07-30
categories: Dockers 
background: '/img/posts/be-linux.jpg.jpg'
---


# Running the CSM Software with Docker
## General Background
For full access to the source code and documentation, visit the GitHub repository: 
https://github.com/continuous-symmetry-measure/csm

The CSM (Continuous Symmetry Measure) software calculates continuous symmetry and chirality measures of molecules with respect to a given symmetry point group G.
This guide explains how to install the CSM without traditional software installation, by utilizing a tool called Docker.

## What is Docker?
Docker is a platform that allows software to run inside isolated "containers", self-contained environments that include all the dependencies and files the software needs. Instead of installing CSM manually, you can use a prebuilt Docker container that comes ready to run.
Advantages:
- Avoid conflicts with other software.
- Ensure consistency and reproducibility.
- Easily share the same setup with colleagues.
Disadvantages:
- The computing power available via Docker on a personal machine may be limited, especially for large-scale or parallel analyses.

## Prerequisites
- Operating system: Linux, macOS, or Windows
- Docker is installed on your machine:
  Download here: https://docs.docker.com/get-docker/
- Docker system requirements are specified here:
  *	https://docs.docker.com/desktop/setup/install/linux/
  *	https://docs.docker.com/desktop/setup/install/mac-install/
  *	https://docs.docker.com/desktop/setup/install/windows-install/

## Docker Workflow
Using Docker involves three key concepts that ensure consistency across different machines:
1. Image - a preconfigured template with everything needed to run the CSM software.
2. Container - an isolated runtime environment created from the image.
3. Execution - running the container with specific CSM parameters to analyze molecules.

### Install the CSM within Docker:
1. Pull (download) the CSM image from Docker Hub. Docker creates a container based on this image.
2. Run the container, passing in molecular files and parameters.

### Verification:
- Check Docker installation: `docker --version`
- Verify Docker is running properly: `docker info`
- List downloaded images: `docker images`
- Test basic container execution and get the CSM help:
  ```bash
  docker run --rm teamcsm/csm:v1.3.7b1 csm --help
  ```
### Detailed Installation Steps
## Step 1: Download the CSM Docker Container
### Note: Working Behind a Proxy
  If you're working in an environment with a network proxy (e.g., a university or institutional firewall), Docker may need to   be configured to access external resources.
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
If your container name is `teamcsm/csm:v1.3.7b1`

You can download the input file for the example from [here](/files/18crown6.mol)
You may want to create an alias:
```bash
alias csm_container_name='docker run -it teamcsm/csm:v1.3.7b1 csm'
```
To analyze a specific molecule (for example, in the C2 point group), use this command:
```bash
# directly
docker run -it teamcsm/csm:v1.3.7b1 csm exact c2 --input 18crown6.mol --output c2-results --keep-structure --remove-hy
# or using alias:
csm_container_name exact c2 --input 18crown6.mol --output c2-results --keep-structure --remove-hy
```
<strong> The result of csm should be: <b>0.0048</b></strong>

Another example:
```bash
# directly
docker run -it teamcsm/csm:v1.3.7b1 csm exact c3 --input 18crown6.mol --output c3-results --keep-structure --remove-hy
# or using alias:
csm_container_name exact c3 --input 18crown6.mol --output c3-results --keep-structure --remove-hy
```
<strong> The result of csm should be: <b>10.2635</b></strong>

Explanation:
- `exact` - Use the exact algorithm for CSM calculation.
- `C2` - The desired point group.
- `--input` - Input molecular file (accepted formats: SDF, XYZ, MOL, PDB).
- `--output` - Output directory where results will be saved.
- `--keep-structure` - Use the structure preserving permutation algorithm to maintain the bonding structure of the molecule. This requires connectivity data in the molecular files.
- `remove-hy` - Ignore the Hydrogen atoms in order to keep the calculation faster.

See all available commands here: https://github.com/continuous-symmetry-measure/csm
  
## Step 3: Access Local Files (Recommended)
To allow the software to read/write files from your computer, it's recommended to map a local folder into the container. Examples:
```bash
docker run -it -v $(pwd):/data teamcsm/csm:v1.3.7b1 csm exact C2 --input /data/molecule.sdf --output /data/results
```
This command maps your current directory to /data inside the container.

## Useful Links
- GitHub Repository: https://github.com/continuous-symmetry-measure/csm
- Docker Hub Page: https://hub.docker.com/r/teamcsm/csm
- Online CSM Calculators: https://csm.ouproj.org.il

## Questions and Support
- Submit issues via the GitHub Issues page.
- Join the Google Group for discussions: https://groups.google.com/g/csm-openu

## Citations
For academic use, please cite the relevant CSM publications as listed in the GitHub README at: 
https://github.com/continuous-symmetry-measure/csm.


