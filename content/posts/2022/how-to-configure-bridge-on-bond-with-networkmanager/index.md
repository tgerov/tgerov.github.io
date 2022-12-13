+++
title = "How to Configure Bridge on Bond With NetworkManager"
date = "2022-12-13T16:42:19+02:00"
author = "Tsvetan Gerov"
authorTwitter = ""
cover = ""
tags = ["linux", "bridge", "bond", "rhel", "networkmanager"]
keywords = ["linux", "bridge", "bond"]
description = "Configure bridge on top of a bonded interface using nmcli."
showFullContent = false
readingTime = false
hideComments = false
+++

I am using RHEL 8 to create and configure network settings, but the same steps should work on RHEL 9 and Clones.

# Network diagram

```
eth0 ---|
        | --+-- bond0 --+-- br0
eth1 ---|
```

# Create bridge
```bash
nmcli connection add type bridge \
    ifname br0 \
    ipv4.method manual \
    ipv4.addresses 192.168.0.40/24 \
    ipv4.gateway 192.168.0.1 \
    ipv4.dns "192.168.0.1" \
    ipv6.method ignore \
    ipv6.never-default true \
    bridge.stp no \
    con-name br0
```

I use a static IP address for my bridge. You will need to adjust the settings to match your needs.

# Create bond
```bash
nmcli connection add type bond \
    ifname bond0 \
    bond.options "mode=active-backup" \
    ipv4.method disabled \
    ipv4.never-default true \
    ipv6.method ignore \
    ipv6.never-default true \
    802-3-ethernet.mtu 1500 \
    con-name bond0 \
    master br0
```

# Add slaves to bond
```bash
nmcli connection add type ethernet ifname eth0 con-name eth0 master bond0
nmcli connection add type ethernet ifname eth1 con-name eth1 master bond0
```
