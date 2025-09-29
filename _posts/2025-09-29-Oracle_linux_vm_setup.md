---
layout: post
title:  "Preparing Oracle Linux VirtualBox for Oracle Database Installation"
author: "Sagiv Barhoom"
date:   2025-09-29
categories: ORACLE,Linux 
background: '/img/posts/be-linux.jpg.jpg'
---

# Preparing Oracle Linux VirtualBox for Oracle Database Installation

This guide summarizes the essential steps to prepare an **Oracle Linux 8.8** virtual machine on **VirtualBox** for installing **Oracle Database 19c**.

## 1. Create the `oracle` User

After the Linux installation, create a dedicated OS user `oracle` and mark him as administrator. This is equivalent to running the following command:

```bash
useradd -m -g oinstall -G dba,oper oracle
passwd oracle
usermod -aG wheel oracle
```

By adding it to the `wheel` group, the user can act as an administrator using `sudo`.

## 2. Set a Hostname

Assign a clear hostname to make the VM easier to identify:

```bash
sudo hostnamectl set-hostname OracleLinux-19c
```

Verify with:

```bash
hostnamectl
```

## 3. Configure Networking

Networking preparation includes three steps:

### 3.1 Configure Bridged Mode

By default, VirtualBox uses **NAT networking**, which restricts access to the VM. This change must be done while the VM is powered off.\
Switch the adapter to **Bridged Mode**, mapping it to your host machine’s physical network card. This allows direct access from the host and other devices on the LAN.

### 3.2 Assign a Static IP Address

Oracle clients need a consistent address. Use `nmcli` (the NetworkManager command‑line tool) to configure a static IP:

```bash
nmcli con modify "static-enp0s3"   ipv4.method manual   ipv4.addresses 192.168.1.99/24   ipv4.gateway 192.168.1.1   ipv4.dns "192.168.1.1 8.8.8.8"   connection.autoconnect yes
nmcli con up "static-enp0s3"
```

Verify the configuration:

```bash
ip addr show enp0s3
```

Expected result:

```
inet 192.168.1.99/24 scope global enp0s3
```

### 3.3 Hosts File Entry (Optional)

On the host machine, you can add an entry to the `hosts` file for easier name resolution:

```
192.168.1.99   oraclelinux-19c
```

## 4. Test Connectivity

- From inside the VM:
  - Test internet access with `ping 8.8.8.8`
  - Test DNS with `ping google.com`
- From the host machine:
  - Confirm connectivity with `ping 192.168.1.99`
- If the hosts file entry was added, verify you can reach `oracledb19.local`.

## 5. Enable SSH Service

To allow file transfer (SCP, WinSCP) and remote access using PuTTY, ensure the SSH service is installed and running:

```bash
sudo dnf install -y openssh-server
sudo systemctl enable sshd
sudo systemctl start sshd
```

Verify the service:
```bash
systemctl status sshd
```

Once enabled, you can connect to the VM using its IP address with any SSH client.

### Test SSH Connectivity and File Transfer
From the host, try logging in:
```bash
ssh oracle@192.168.1.99
```
If successful, test file transfer with `scp`:
```bash
scp testfile.txt oracle@192.168.1.99:/home/oracle/
```
Confirm on the VM that the file arrived in `/home/oracle/`.

## Final State

Your Oracle Linux VM is now:

- Equipped with a dedicated `oracle` user
- Assigned a custom **hostname**
- Connected to the LAN via **Bridged Adapter**
- Configured with a **static IP (192.168.1.99)**
- Ready for both network and DNS connectivity

The environment is ready for the next stage: downloading and installing **Oracle Database 19c**.
