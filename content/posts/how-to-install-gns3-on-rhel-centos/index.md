---
categories:
- Linux
date: "2020-04-23T12:13:39+03:00"
draft: false
tags:
- linux
- gns3
- rhel
title: How to install GNS3 on RHEL 8 / CentOS 8
summary: Install GNS3 on RHEL 8/CentOS 8 with Dynamips, VPCS and QEMU.
---

## What is GNS3 ?
Graphical Network Simulator-3 (shortened to GNS3) is a network software emulator first released in 2008.
It allows the combination of virtual and real devices, used to simulate complex networks. 
It uses Dynamips emulation software to simulate Cisco IOS. [Source - WikiPedia](https://en.wikipedia.org/wiki/Graphical_Network_Simulator-3).

## How to install GNS3
I will show you how to install GNS3, dynamips, vpcs and ubridge from official GNS3 source code on RHEL 8 / CentOS 8.

The only difference in the installation process between RHEL and CentOS is [Code Ready Linux Builder](https://developers.redhat.com/blog/2018/11/15/introducing-codeready-linux-builder/) repository, which is called [PowerTools](https://wiki.centos.org/FAQ/CentOS8#Where_is_the_CentOS_8_codeready-developer_equivalent_repo.3F) on CentOS.

### Enable CodeReady (RHEL 8 Only)
````bash
[root@rh ~]# subscription-manager repos --enable codeready-builder-for-rhel-8-x86_64-rpms
````
### Enable PowerTools (CentOS 8 Only) 
````bash
[root@centos ~]# yum install dnf-plugins-core
[root@centos ~]# yum config-manager --set-enabled PowerTools
````
### Install Required RPMs
````bash
[root@rh ~]# yum install python3-devel python3-pip cmake make elfutils-libelf-devel libpcap-devel python3-pyqt5-sip python3-qt5 xterm
[root@rh ~]# yum groupinstall 'Development Tools'
````
### Clone the needed sources for gns3-gui, gns3-server, vpcs, dynamips and ubridge
````bash
[root@rh ~]# cd /usr/local/src/
[root@rh src]# git clone https://github.com/GNS3/gns3-server.git
[root@rh src]# git clone https://github.com/GNS3/gns3-gui.git
[root@rh src]# git clone https://github.com/GNS3/vpcs.git
[root@rh src]# git clone https://github.com/GNS3/dynamips.git
[root@rh src]# git clone https://github.com/GNS3/ubridge.git
````
### Install GNS3 Server
````bash
[root@rh ~]# cd /usr/local/src/gns3-server/
[root@rh gns3-server]# pip3 install -r requirements.txt
[root@rh gns3-server]# python3 setup.py install
````
### Install GNS3 Gui
````bash
[root@rh ~]# cd /usr/local/src/gns3-gui/
[root@rh gns3-gui]# pip3 install -r requirements.txt
[root@rh gns3-gui]# python3 setup.py install
[root@rh gns3-gui]# cp resources/linux/applications/gns3.desktop /usr/share/applications/
[root@rh gns3-gui]# cp -R resources/linux/icons/hicolor/ /usr/share/icons/

````
### Install VPCS
````bash
[root@rh ~]# cd /usr/local/src/vpcs/src
[root@rh src]# ./mk.sh
[root@rh src]# cp vpcs /usr/local/bin/vpcs
````
### Install Dynamips
````bash
[root@rh ~]# cd /usr/local/src/dynamips/
[root@rh dynamips]# mkdir build 
[root@rh dynamips]# cd build/
[root@rh build]# cmake ..
[root@rh build]# make
[root@rh build]# make install
````
### Install Ubridge
````bash
[root@rh ~]# cd /usr/local/src/ubridge
[root@rh ubridge]# make
[root@rh ubridge]# make install
````

Now GNS3 will be listed in your applications.
Optionally, libvirt and qemu-kvm can be installed and it will be possible to add virtual machines directly in GNS3 topology.

````bash
[root@rh ~]# yum install libvirt qemu-kvm
````

Copyright 2021 Tsvetan Gerov | <a href=https://creativecommons.org/licenses/by-sa/2.0/ target=_blank rel=noopener>CC BY-SA 2.0</a>

