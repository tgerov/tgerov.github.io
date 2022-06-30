---
categories:
- Linux
date: "2021-03-18T17:56:35+02:00"
draft: false
tags:
- linux
- veeam
- openvz
title: How to install Veeam Agent on OpenVZ 7
summary: Deploy Veeam Agent 5 on OpenVZ 7
---

## Download latest EL7 RPMs from Veeam [repository](http://repository.veeam.com/backup/linux/agent/rpm/el/7/x86_64/).
````bash
[root@openvz7 ~] # wget http://repository.veeam.com/backup/linux/agent/rpm/el/7/x86_64/veeam-5.0.0.4318-1.el7.x86_64.rpm
[root@openvz7 ~] # wget http://repository.veeam.com/backup/linux/agent/rpm/el/7/x86_64/veeamsnap-5.0.0.4318-1.noarch.rpm
````

## Install the required dependencies
````bash
[root@openvz7 ~] # yum install dkms vzkernel-headers-$(uname -r) vzkernel-devel-$(uname -r)
````

## Install VeeamSnap and wait for the kernel module to be compiled
````bash
[root@openvz7 ~] # rpm -ivh veeamsnap-5.0.0.4318-1.noarch.rpm
````

## Load veeamsnap kernel module
````bash
[root@openvz7 ~] # modprobe veeamsnap
````

## You can verify that the module is lodaded with lsmod
````bash
[root@openvz7 ~]# lsmod | grep veeamsnap
veeamsnap             179802  0 
````

## Install Veeam Agent
````bash
[root@openvz7 ~]# rpm -ivh veeam-5.0.0.4318-1.el7.x86_64.rpm
````

## Verify that the service is running
````bash
[root@openvz7 ~]# systemctl status veeamservice
● veeamservice.service - Veeam Agent for Linux service daemon
   Loaded: loaded (/usr/lib/systemd/system/veeamservice.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2021-03-10 07:14:46 EST; 10s ago
  Process: 521861 ExecStart=/usr/sbin/veeamservice --daemonize --pidfile=${PIDFILE} (code=exited, status=0/SUCCESS)
 Main PID: 521904 (veeamservice)
    Tasks: 45
   CGroup: /system.slice/veeamservice.service
           └─521904 /usr/sbin/veeamservice --pidfile /var/run/veeamservice.pid --daemon
````

Now, you can add your OpenVZ server in Veeam Backup and Replication.

 
Copyright 2021 Tsvetan Gerov | <a href=https://creativecommons.org/licenses/by-sa/2.0/ target=_blank rel=noopener>CC BY-SA 2.0</a>

