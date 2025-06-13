
# 🖧 DNS Server Setup on Ubuntu VM with Windows Client

This repository provides step-by-step documentation to set up a DNS server using BIND9 on an Ubuntu VM.
The setup allows a Windows machine to resolve custom domain names via the DNS server.


## 📚 Table of Contents
```bash

- [System Roles and IP Plan](#system-roles-and-ip-plan)
- [Installation](#installation)
- [DNS Zone Configuration](#dns-zone-configuration)
- [Netplan Static IP Configuration](#netplan-static-ip-configuration)
- [Client Configuration](#client-configuration)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)
- [Progress Log](#progress-log)
- [Conclusion](#conclusion)
```

DNS Server Setup on Ubuntu VM (with Windows Client in LAN)
Objective
This guide walks you through how to set up a DNS server on an Ubuntu Virtual Machine (VM) hosted on a Windows system. The Ubuntu VM will serve domain name resolution requests for a Windows client over a local network (LAN) using custom domain names like www.ajaydns.local.
System Roles and IP Plan
Server (Ubuntu VM): 192.168.18.8 – Hosts the DNS server (BIND9)
Client (Windows): 192.168.18.6 – Resolves names using DNS server
Ensure both systems are on the same bridged network to communicate directly.

### Step 1: Install BIND9 DNS Server

### Install the BIND9 package and utilities:
```bash
sudo apt update
sudo apt install bind9 bind9utils bind9-doc dnsutils

### Step 2: Create the Forward Zone File

### Forward zone maps domain names to IP addresses. Create and edit /etc/bind/db.ajaydns.local:
$TTL 604800
@ IN SOA ns.ajaydns.local. root.ajaydns.local. (
2 ; Serial
604800 ; Refresh
86400 ; Retry
2419200 ; Expire
604800 ) ; Negative Cache TTL
@ IN NS ns.ajaydns.local.
ns IN A 192.168.18.8
www IN A 192.168.18.8
windows IN A 192.168.18.6

### Step 3: Create the Reverse Zone File

### Reverse zone maps IP addresses to hostnames. Create and edit /etc/bind/db.192:
$TTL 604800
@ IN SOA ns.ajaydns.local. root.ajaydns.local. (
2
604800
86400
2419200
604800 )
@ IN NS ns.ajaydns.local.
8 IN PTR ns.ajaydns.local.
20 IN PTR www.ajaydns.local.
6 IN PTR windows.ajaydns.local.

### Step 4: Declare Zones in named.conf.local

### Edit /etc/bind/named.conf.local:
zone "ajaydns.local" {
type master;
file "/etc/bind/db.ajaydns.local";
};
```

```bash
zone "18.168.192.in-addr.arpa" {
type master;
file "/etc/bind/db.192";
};

### Step 5: Validate Zone File Syntax

### Run these commands to check for syntax errors:
sudo named-checkzone ajaydns.local /etc/bind/db.ajaydns.local
sudo named-checkzone 18.168.192.in-addr.arpa /etc/bind/db.192
sudo named-checkconf

### Step 6: Restart BIND9

### Restart the service to apply changes:
sudo systemctl restart bind9

### Check status with:
sudo systemctl status bind9
Optional: Configure Static IP using Netplan

### Edit /etc/netplan/01-netcfg.yaml:

### network:
version: 2
renderer: networkd

### ethernets:

### enp0s3:
dhcp4: no

### addresses:
- 192.168.18.8/24

### nameservers:
addresses: [192.168.18.8]

### routes:
- to: default
via: 192.168.18.1
Apply with: sudo netplan apply
Client Configuration (Windows)
Set static IP: 192.168.18.6, Gateway: 192.168.18.1, DNS: 192.168.18.8

### Test with Command Prompt:
nslookup www.ajaydns.local
ping www.ajaydns.local
Verification from Ubuntu

### Use nslookup to test from server:
nslookup www.ajaydns.local 127.0.0.1
nslookup windows.ajaydns.local 127.0.0.1

### DNS Resolution and Connectivity Test
```

The following screenshot shows the execution of `nslookup` and `ping` commands to verify DNS configuration and network connectivity.

![DNS Test Screenshot](https://github.com/user-attachments/assets/ba70e70c-0efe-442b-9034-57474df154b1)
![DNS Test Screenshot](https://github.com/user-attachments/assets/ba70e70c-0efe-442b-9034-57474df154b1)

Common Issues & Fixes
Destination host unreachable → Check LAN connectivity
Connection refused → Restart and check BIND9
Wrong DNS response → Use named-checkzone
Netplan warnings → Set file permissions to 600 and use 'routes' instead of 'gateway4'
Optional: Add More Clients
Add entries to forward and reverse zone files for new clients.
Conclusion
You now have a working DNS server setup on Ubuntu VM serving custom domain names to a Windows client in LAN. This setup is ideal for learning, testing, and internal networking purposes.
