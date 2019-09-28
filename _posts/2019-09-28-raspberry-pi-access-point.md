---
layout: post
title: "Raspberry Pi Access Point"
date: 2019-09-28 11:02:52 -0600
categories: blog
---
# Raspberry Pi Access Point Setup

- [Prerequisites](#prerequisites)
  - [Update System](#update-system)
  - [Install hostapd and dnsmasq](#install-hostapd-and-dnsmasq)
- [Configure Access Point](#configure-access-point)
  - [Edit dhcpcd.conf](#edit-dhcpcd.conf)
  - [Replace dnsmasq.conf](#replace-dnsmasq.conf)
  - [Create hostapd.conf](#create-hostapd.conf)
  - [Edit hostapd](#edit-hostapd)
  - [Create Access Point Service](#create-access-point-service)
    - [Create the script](#create-the-script)
    - [Create the service](#create-the-service)
  - [Permissions](#permissions)
  - [Modify sysctl.conf](#modify-sysctl.conf)
  - [Configure Services](#configure-services)
  - [Reboot](#reboot)
- [Debugging](#debugging)
- [Resources](#resources)

## Prerequisites

Before getting started, make sure that the system is updated and the necessary software is installed.

### Update System

Run apt-get update and upgrade to make sure you have the latest and greatest bits.

```bash
sudo apt-get update
sudo apt-get upgrade
```

### Install hostapd and dnsmasq

Install the hostapd access point daemon and the dnsmasq dhcp service.

```bash
sudo apt-get -y install hostapd dnsmasq
```

## Configure Access Point

Here we need to edit the config files for dhcpcd, hostapd, and dnsmasq so that they all play nice together.
***Do NOT, as in past implementations, make any edits to the `/etc/network/interfaces` file. This can cause problems, per tutorial notes [here](https://raspberrypi.stackexchange.com/questions/37920/how-do-i-set-up-networking-wifi-static-ip-address/37921#37921)***

### Edit dhcpcd.conf

Edit and add the following lines in `/etc/dhcpcd.conf`. This sets up a static IP address on the uap0 interface that we will set up in the startup script. The nohook line prevents the 10-wpa-supplicant hook from running wpa-supplicant on this interface.

***You can set the `4` in the IP address to whatever you would like, but keep track of it as we will use it again later***

```bash
interface uap0
    static ip_address=192.168.4.1/24
    nohook wpa_supplicant
```

### Replace dnsmasq.conf

Save a copy of `/etc/dnsmasq.conf` as it is a useful example, you may even want to use some of the RPi-specific lines at the end.

```bash
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
```

Create a new `/etc/dnsmasq.conf`. This file handles the IP addresses of the devices that connect to its access point:

```bash
sudo vi /etc/dnsmasq.conf
```

Add the following to it:

```bash
interface=lo,uap0               #Use interfaces lo and uap0
bind-interfaces                 #Bind to the interfaces
server=8.8.8.8                  #Forward DNS requests to Google DNS
domain-needed                   #Don't forward short names
bogus-priv                      #Never forward addresses in the non-routed address spaces
# Assign IP addresses between 192.168.4.50 and 192.168.4.150 with a 12-hour lease time
dhcp-range=192.168.4.50,192.168.4.150,12h
```

***Replace the `4` if you made it different in the previous step***

### Create hostapd.conf

Create file `/etc/hostapd/hostapd.conf`. This file handles the configuration and broadcasting of the access point including the SSID and password:

```bash
sudo vi /etc/hostapd/hostapd.conf
```

 Add the following:

***Change `ssid` and `wpa_passphrase` to whatever you desire them to be***

```bash
interface=uap0
ssid=pi
wpa_passphrase=raspberry
channel=1
hw_mode=g
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
driver=nl80211
```

*Note: The channel written here **MUST** match the channel of the wifi that you connect to in client mode (via wpa-supplicant). If the channels for your AP and STA mode services do not match, then one or both of them will not run. This is because there is only one physical antenna. It cannot cover two channels at once.*

### Edit hostapd

Edit file `/etc/default/hostapd`. This file is the base configuration file for `Hostapd`, we will update it to use the configuration file we just created:

Replace

```bash
#DAEMON_CONF
```

with

```bash
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

### Create Access Point Service

In order to use the access point in the most efficient way, we will create an access point service. This will allow the access point to quickly and easily be started, stopped, or restarted. And it also handles restart on failure in case something goes wrong the first time it tries to start.

#### Create the script

Create the access point script

```bash
sudo vi /usr/bin/accesspoint
```

Then paste in the following code

```bash
#!/bin/bash

echo "Stopping network services (if running)..."
systemctl stop hostapd.service
systemctl stop dnsmasq.service
systemctl stop dhcpcd.service

echo "Removing uap0 interface..."
iw dev uap0 del

echo "Adding uap0 interface..."
iw dev wlan0 interface add uap0 type __ap

echo "Editing IP tables..."
iptables -t nat -A POSTROUTING -s 192.168.4.0/24 ! -d 192.168.4.0/24 -j MASQUERADE

echo "Bringing up uap0..."
ifconfig uap0 up

echo "Starting hostapd service..."
systemctl start hostapd.service
sleep 10

echo "Starting dhcpcd service..."
systemctl start dhcpcd.service
sleep 5

echo "Starting dnsmasq service..."
systemctl start dnsmasq.service
```

Then make sure this is executable with

```bash
sudo chmod +x /usr/bin/accesspoint
```

#### Create the service

Now create the service on the system

```bash
sudo vi /etc/systemd/system/accesspoint.service
```

To edit this file after creation, use the following command:

```bash
sudo systemctl edit --full accesspoint.service
```

Add the following code to the service file:

```bash
[Unit]
Description=AccessPoint

[Service]
ExecStart=/usr/bin/accesspoint
Type=simple
RemainAfterExit=yes
Restart=on-failuer
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=accesspoint
User=pi
Group=pi

[Install]
WantedBy=multi-user.target
```

*If editing a service, you may need to run `sudo systemctl daemon-reload` to reload the unit*

### Permissions

In order for the access point to get started and stopped by the default user, edit the sudoers file using

```bash
sudo visudo
```

Add the following to the file:

```bash
pi      ALL=(ALL) NOPASSWD:/sbin/reboot
pi      ALL=(ALL) NOPASSWD:/sbin/iw dev uap0 del
pi      ALL=(ALL) NOPASSWD:/sbin/ifconfig uap0 up
pi      ALL=(ALL) NOPASSWD:/sbin/iptables -t nat -A POSTROUTING -s 192.168.4.0/24 ! -d 192.168.4.0/24 -j MASQUERADE
pi      ALL=(ALL) NOPASSWD:/sbin/iptables -t nat -D POSTROUTING -s 192.168.4.0/24 ! -d 192.168.4.0/24 -j MASQUERADE
pi      ALL=(ALL) NOPASSWD:/sbin/iw dev wlan0 interface add uap0 type __ap
pi      ALL=(ALL) NOPASSWD:/bin/systemctl start accesspoint
pi      ALL=(ALL) NOPASSWD:/bin/systemctl stop accesspoint
pi      ALL=(ALL) NOPASSWD:/bin/systemctl enable accesspoint
pi      ALL=(ALL) NOPASSWD:/bin/systemctl disable accesspoint
pi      ALL=(ALL) NOPASSWD:/bin/systemctl restart dhcpcd
pi      ALL=(ALL) NOPASSWD:/bin/systemctl start dnsmasq
pi      ALL=(ALL) NOPASSWD:/bin/systemctl reload dnsmasq
pi      ALL=(ALL) NOPASSWD:/bin/systemctl stop dnsmasq
pi      ALL=(ALL) NOPASSWD:/bin/systemctl start hostapd
pi      ALL=(ALL) NOPASSWD:/bin/systemctl stop hostapd
pi      ALL=(ALL) NOPASSWD:/bin/systemctl enable hostapd
pi      ALL=(ALL) NOPASSWD:/bin/systemctl disable hostapd
```

### Modify sysctl.conf

Edit the `sysctl` configuration file to enable IPv4 packet forwarding. This allows the device you connect to the Raspberry Pi to have internet access using an Ethernet or WiFi connection on the Pi.

```bash
sudo vi /etc/sysctl.conf
```

Uncomment the next line to enable packet forwarding for IPv4

```bash
net.ipv4.ip_forward=1
```

### Configure Services

#### Enable these services

```bash
sudo systemctl enable accesspoint.service
```

```bash
sudo systemctl enable dhcpcd.service
```

#### Disable these services

```bash
sudo systemctl disable hostapd.service
```

```bash
sudo systemctl disable dnsmasq.service
```

### Reboot

Now that all of the network settings are configured, reboot the pi.

```bash
sudo reboot
```

## Starting and Stopping the Access Point

In order to start or stop the access point or to enable or disable it on startup, run one the following commands

```bash
# Stop access point
sudo systemctl stop accesspoint

# Start access point
sudo systemctl start accesspoint

# Enable access point on startup
sudo systemctl enable accesspoint

# Disable access point on startup
sudo sudo systemctl disable accesspoint
```

## Debugging

If the network is not working or the access point isn't working check the following services

```bash
# Check status of access point service
sudo systemctl status accesspoint.service

# Check status of hostapd service
sudo systemctl status hostapd.service

# Check status of dnsmasq service
sudo systemctl status dnsmasq.service

# Check status of dhcpcd service
sudo systemctl status dhcpcd.service
```

You can also watch all of the logs of the access point using the following command

```bash
journalctl -fu accesspoint -fu hostapd -fu dnsmasq -fu dhcpcd
```

## Resources

- [Raspberry Pi Forum](https://lb.raspberrypi.org/forums/viewtopic.php?t=211542#p1355569)
- [Sparkfun Tutorial](https://learn.sparkfun.com/tutorials/setting-up-a-raspberry-pi-3-as-an-access-point/all)
- [Raspap Webgui](https://github.com/billz/raspap-webgui)
- [Another Raspberry Pi Forum](https://www.raspberrypi.org/forums/viewtopic.php?f=36&t=138730&sid=fe5fa62f0fbc726ced9cbf5a0c5a65ff#p938306)
