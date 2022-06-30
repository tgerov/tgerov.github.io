---
author: Tsvetan Gerov
categories:
- Linux
date: "2022-02-12"
summary: Unlock LUKS encrypted root file system remotely via SSH
tags:
- linux
- rhel
- centos
- luks
title: Unlock LUKs encrypted root filesystem remotely via SSH (RHEL 8/CentOS 8)
---

## Intruduction
In this short tutorial, I will show you how to unlock your luks encrypted root file system on RHEL 8 / CentOS 8, remotely via SSH. To accomplish this task, we will use 3rd party dracut module - dracut-sshd.

Before we begin, we will need some details for our system - Ethernet device, IP address, NETMASK and Gateway. 

In this example my configuration is as follow:

IP: 192.168.0.77  
NETMASK: 255.255.0.0  
GATEWAY: 192.168.0.1  
Ethernet: eth0  



## Generate SSH key pair
We need to generate a public-private ssh keys and include them in initramfs.

```bash
ssh-keygen -t rsa 
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.0.77
```

## Install dracut-sshd
This Dracut module (dracut-sshd) integrates the OpenSSH sshd into the initramfs. It allows for remote unlocking of a fully encrypted root filesystem and remote access to the Dracut emergency shell (i.e. early userspace). [1]

```bash
dnf copr enable gsauthof/dracut-sshd
dnf install dracut-sshd
```

## Enable dracut Network

Here, we add early network capabilities to initramfs. 
Update network configuration to match your requirements.

```bash
cat <<EOF  >> /etc/dracut.conf.d/99-network.conf
kernel_cmdline="ip=192.168.0.77::192.168.0.1:255.255.0.0:VerySecureVM:eth0:off"
EOF
```

Syntax is:
```
ip<client-IP-number>:[<server-id>]:<gateway-IP-number>:<netmask>:<client-hostname>:<interface>:{dhcp|dhcp6|auto6|on|any|none|off}
```

## Force Dracut initram regeneration
In order to include our SSH keys and dracut network configuration, we will force dracut to regenerate initramfs image.

```bash
dracut -f
```

## Flush Dracut network configuration

Since IP address configuration is set via kernel, NetworkManager will not load your configuration from real root filesystem. This will be a problem if you have IP aliasess, bonds, bridge, vlans & etc. configured on your real root.

If your system has one IP address, you can skip these steps.

```bash
cat <<EOF >> /etc/systemd/system/flush-dracut-network\@.service
[Unit]
Description=Remove dracut's network configuration for %I
Before=network-pre.target
Wants=network-pre.target

[Service]
ExecStartPre=/usr/sbin/ip address show %i
ExecStart=/usr/sbin/ip -statistics address flush dev %i

[Install]
WantedBy=default.target
EOF

```

 Enable systemd unit to flush dracut configuration on our ethernet interface.
 
```bash
systemctl enable flush-dracut-network@eth0
```

Reboot your machine and test : )

Once the initrd load and network is fired up, you should be able to login via SSH and unlock your encrypted root file system with single command - systemd-tty-ask-password-agent. Here is an example:

```bash
ssh root@192.168.0.77
initramfs-ssh:/root# systemd-tty-ask-password-agent
Please enter passphrase for disk QEMU_HARDDISK (luks-01a6fbf5-779e-47c0-b7bd-efca0252d5d9)! ********
initramfs-ssh:/root# Connection to 192.168.0.77 closed by remote host.
Connection to 192.168.0.77 closed.
```

References:  
[1] https://github.com/gsauthof/dracut-sshd  
[2] https://access.redhat.com/solutions/3017441  
[3] https://unix.stackexchange.com/questions/506331/networkmanager-doesnt-change-ip-address-when-dracut-cmdline-provided-static-ip  
[4] Photo: [Markus Winkler](https://unsplash.com/photos/SZ98vfIx0pw)
