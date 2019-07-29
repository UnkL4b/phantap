# PhanTap (Phantom Tap)

![PhanTap Logo](/img/phantap.png)

PhanTap is an OpenWrt package used as a silent network tap and it does not affect the victim’s traffic, even in networks having NAC (Network Access Control 802.1X - 2004).
PhanTap will analyze traffic on the network and mask its traffic as the victim device.
It can mount a tunnel back to a remote server, giving the user a foothold in the network for further analysis and pivoting.
The physical device used for our testing is currently a small, inexpensive router, the GL.iNet GL-AR150.


## Features:

* Transparent network bridge.
* Silent : no arp, multicast, broadcast.
* 802.1x passthrough.
* Automatic configuration:
    * capture traffic exiting the network (the destination is non RFC1918), source IP and MAC is our victim, destination MAC is our gateway,
    * SNAT bridge traffic to the victim MAC and IP address,
    * set the router default gateway to the MAC of the gateway detected just before.
* Introspects ARP, multicast and broadcast traffic and adds a route to the machine IP address and adds the machine MAC address to the neighbor list, hence giving the possibility of talking to all the machines in the local network.
* Learns the DNS server from traffic and modifies the one on the router so that it's the same.
* Can run commands (ex: /etc/init.d/openvpn restart) when a new IP is configured.
* Lets you choose any VPN software, for example OpenVPN tcp port 443 so it goes through most firewalls.

## Setup

(This assumes phantap packages are already part of the OpenWrt packages feed)

* Install PhanTap package:
```
opkg install phantap phantap-learn
```
* Configure the Wifi and start administering the router through it.
* Either reboot the device, or run `/etc/init.d/phantap setup`.
* Get the interface names from that device:
```
# uci show network | grep ifname
network.loopback.ifname='lo'
network.lan.ifname='eth1'
network.wan.ifname='eth0'
network.wan6.ifname='eth0'
```
In this example we are using a GL-AR150, which only has 2 interfaces.

* Add the interfaces to the phantap bridge via the following commands in the cli
(assuming we are using a GL-AR150):
```
uci delete network.lan.ifname
uci delete network.wan.ifname
uci delete network.wan6.ifname
uci set network.phantap.ifname='eth0 eth1'
uci commit network
/etc/init.d/network reload
```

* Phantap is now configured, as soon as you plug it between a victim and their switch, it will automatically configure the router and give it Internet access.

* You can add your favorite VPN to have a remote connection back.
* You can also look at disabling the wifi by default and using hardware buttons to start it (https://openwrt.org/docs/guide-user/hardware/hardware.button).

## Limitations or how it can be detected :

* The GL.iNet GL-AR150 and most inexpensive devices only support 100Mbps, meanwhile modern network traffic will be 1Gbps.
* The network port  will stay up, switch side, when the victim device is disconnected/shutdown.
* There is no re-configuration of PhanTap, so we might use an IP that has been reattributed to another device (roadmap DHCP).
* Some traffic is blocked by the Linux bridge (STP/Pause frames/LACP).
* You cannot talk to the victim machine, as you use its IP.

## Roadmap :

* Add logic to restart the detection when the links go up/down.
* Add DHCP packet analysis for dynamic reconfiguration.
* Add IPv6 support.
* Test limitations of devices that have switches(swconfig) instead of separate interfaces.