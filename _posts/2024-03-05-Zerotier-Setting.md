---
title: Zerotier route between two networks
date: 2024-03-05 11:31:00 -0700
categories: [Zerotier]
tags: 
Pin:
---

### Introduction
I have been using Zerotier for many years, and it has been very helpful. However, installing it on every device is inconvenient. I am trying to set up a route between two networks. I have networks A and B, each with a device running Ubuntu. I want other devices in networks A and B to access each other through these Ubuntu devices. The same setup should apply if I have additional networks, such as C or D.

### Step 1: Install Zerotier
To install Zerotier on Ubuntu, you can access to https://www.zerotier.com/download/ 
I am useing docker to install Zerotier on Ubuntu.
```shell
docker pull zerotier/zerotier
docker run --name myzerotier -d --rm --cap-add NET_ADMIN --device /dev/net/tun --net=host zerotier/zerotier:latest YOU_ZEROTIER_NETWORK_ID 
# Start the Zerotier service and join the network
```

### Step 2: Configure Zerotier
You can login to the Zerotier web interface to configure the network. Add the device to the network and set the IP address of the device.

### Step 3: Mutual access between the Local Area Network(LAN) and Zerotier nodes
You should do the following steps:
- Add the routes in the zerotier website. Destination LAN ip 192.168.1.0 via Zerotier ip 172.22.0.6
- Set IP forwarding on the Ubuntu device.
```shell
sudo sysctl -w net.ipv4.ip_forward=1
sudo sysctl -p
```
- Set iptables rules to forward the traffic between the LAN and Zerotier nodes.
```shell
sudo iptables -I FORWARD -i eth0 -j ACCEPT
sudo iptables -I FORWARD -o eth0 -j ACCEPT
sudo iptables -t nat -I POSTROUTING -o eth0 -j MASQUERADE

sudo iptables -I FORWARD -i ZEROTIER_NETWORK_ID -j ACCEPT
sudo iptables -I FORWARD -o ZEROTIER_NETWORK_ID -j ACCEPT
sudo iptables -t nat -I POSTROUTING -o ZEROTIER_NETWORK_ID -j MASQUERADE
# Replace ZEROTIER_NETWORK_ID with the network ID of the Zerotier network. 'ip a' can get it.
```
- Add static routes to the Ubuntu device. Like 172.22.0.0 via 192.168.1.6(Ubuntun IP)
- Save the iptables rules.
```shell
sudo apt install iptables-persistent
sudo netfilter-persistent save
```