---
categories:
- Linux
date: "2021-03-18T18:20:49+02:00"
draft: false
lastmod: "2022-05-28T18:23:49+02:00"
tags:
- linux
- gns3
- fedora
title: How to install GNS3 on Fedora
summary: Install GNS3 on Fedora with Dynamips, VPCS and QEMU.
---


## What is GNS3 ?
Graphical Network Simulator-3 (shortened to GNS3) is a network software emulator first released in 2008.
It allows the combination of virtual and real devices, used to simulate complex networks. 
It uses Dynamips emulation software to simulate Cisco IOS ([WikiPedia](https://en.wikipedia.org/wiki/Graphical_Network_Simulator-3)).


## How to install GNS3
I will show you how to install GNS3, dynamips, vpcs and ubridge on Fedora 36/35/34.

## Enable RPMFusion repositories
Note: Non-free RPMFusion repository is required for dynamips. If you don't need it, you can skip this step.

````bash
  $ sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
````

## Enable my copr repository with VPCS
Note: My VPCS repository is required because VPCS is not available in official Fedora repositories. If you do not need VPCS, you can skip this step. 

````bash
  $ sudo dnf copr enable tgerov/vpcs
````

# You can now perform the actual installation of GNS3
````bash
  $ sudo dnf install gns3-gui gns3-server dynamips vpcs ubridge
````

Now GNS3 will be listed in your applications.
Optionally, libvirt and qemu-kvm can be installed and it will be possible to add virtual machines directly in GNS3 topology.

````bash
  $ sudo dnf install libvirt qemu-kvm
  $ sudo gpasswd -a <My-User> libvirt
````

Copyright 2021 Tsvetan Gerov | <a href=https://creativecommons.org/licenses/by-sa/2.0/ target=_blank rel=noopener>CC BY-SA 2.0</a>

