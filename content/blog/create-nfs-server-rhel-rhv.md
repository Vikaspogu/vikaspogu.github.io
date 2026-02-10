---
title: "Create and Add NFS Storage to RHV"
date: 2019-11-02
description: "Configure NFS storage domains on Red Hat Virtualization using a RHEL server with proper exports and firewall rules."
tags: ["RHV", "NFS", "Storage", "Linux"]
slug: "nfs-storage-rhv"
---

Set up NFS shares that will serve as storage domains on a Red Hat Enterprise Linux server.

Add a firewall rule to RHEL server.

```bash
firewall-cmd --permanent --add-service nfs
firewall-cmd --permanent --add-service mountd
firewall-cmd --reload
```

Create and add group `kvm`; set the ownership of your exported directories to 36:36, which gives vdsm:kvm ownership; change the director's permission so that read and write access is granted to the owner.

```bash
groupadd kvm -g 36
useradd vdsm -u 36 -g 36
mkdir -pv /home/rhv-data-vol
chown -R 36:36 /home/rhv-data-vol
chmod 0755 /home/rhv-data-vol
```

Add newly created directory to `/etc/exports` file, which controls file systems exported to remote hosts and specifies options.

```bash
echo "/home/rhv-data-vol 192.168.10.1(rw,sync)" >> /etc/exports
exportfs -av
```

Start, enable NFS server; checklist of export server using `showmount`

```bash
systemctl start nfs
systemctl enable nfs
systemctl status nfs
showmount -e
```
