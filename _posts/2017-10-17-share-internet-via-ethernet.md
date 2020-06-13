---
title: "Share internet via ethernet cable"
excerpt_separator: "<!--more-->"
categories:
  - networking
tags:
  - bash
  - linux
  - networking
  - internet
---

We all have been in that situation where all you have is a raspberrypi(no wifi), and you got your laptop. But then you dont have an external USB wifi stick. But then you do happen to have an ethernet cable. Your amazing ideas cant come to life unless you can get that pi connected to the internet.

Have no fear! today, i will save you with `dnsmasq` and a little bit of configuration!

### Requirements
All whats required is the ethernet cable. Assuming that you have a laptop rocking linux, and your laptop has wifi & an ethernet port. Also your raspberrypi or which ever device, has power and an ethernet port as well.

### 1. Select an ip-address range.
This ip address range is for all the clients that you will have in the future. I personally went with the private ip address range 10.0.0.0/24.
  * **[10.0.0.0] network address**
  * **[10.0.0.1] ethernet interface address**
  * **[10.0.0.x] client addresses (2 < x < 255)**

### 2. Setup the ethernet interface on the host
The following commands will set the interface up and set an ip on the interface
```shell
$ sudo ip link set dev eth0 up
$ ip addr add 10.7.7.1/24 dev eth0
```

### 3. Install and configure dnsmasq
Dnsmasq is a very powerful lightweight application to provide network infrastructure for small networks. I will be using it here to provide DHCP service for the clients which will connect to the ethernet port.
```shell
# installing the dnsmasq
$ sudo pacman -Syu dnsmasq
# use apt-get for ubuntu or debian

# edit the following file to configure dnsmasq
$ sudo vim /etc/dnsmasq.conf

# @file /etc/dnsmasq.conf
interface=eth0
bind-interfaces
dhcp-range=10.7.7.2,10.7.7.254
#__end of file__#

# to start dnsmasq
$ sudo systemctl start dnsmasq

# to check if dhcp server has released an ip
$ sudo journalctl -u dnsmasq

# or to check for dhcp leased addresses on a log file
$ cat /var/lib/misc/dnsmasq.leases
```

### 4. Setup IP Forwarding and Enable NAT
To allow the pi to have access to the internet via the laptops wifi, we will need to forward the packets from the pi through the wifi interface on the laptop. We will also need to enable NAT since its through a single interface that we allow the pi to connect to the internet. The laptops public IP address will be the pi's public IP address as well.
```shell
# enable ip forwarding [temporary but immediately]
$ sudo sysctl net.ipv4.ip_forward=1
$ sudo sysctl net.ipv4.conf.eth0.forwarding=1

# enable ip forwarding [permanent but require restart]
$ sudo vim /etc/sysctl.d/30-ipforward.conf

# @file /etc/sysctl.d/30-ipforward.conf
net.ipv4.ipforward=1
net.ipv6.conf.default.forwarding=1
net.ipv6.conf.all.forwarding=1
#__end of file__#

# enable Network Address Translation to your wireless
# assuming your wireless device name is wlan0
$ sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
$ sudo iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
```

That should be it! you can check if you have leased an IP address by issuing the command `sudo journalctl -u dnsmasq` or `cat /var/lib/misc/dnsmasq.leases`. After finding out the IP address of your device/pi, ssh into it and `sudo apt-get update` and see the magic happen. One cable, but you can ssh into it and it has internet access!
