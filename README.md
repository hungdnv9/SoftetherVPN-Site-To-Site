## SoftetherVPN:Build a LAN-to-LAN VPN (Using L3 IP Routing)  

## TOPOLOGY

## Network Details

| Roles	               | Network         | Gateway        |
|--------------------- |:---------------:|---------------:| 	
| VPN Client           | 192.168.99.0/24 | 192.168.99.254 |
| VietNam Production   | 10.199.0.0/24   | 10.199.0.249   |
| VietNam Sanbox       | 10.198.0.0/24   | 10.198.0.249   |
| VietNam Dev          | 10.197.0.0/24   | 10.197.0.249   |
| Singapore Production | 10.200.0.0/24   | 10.200.0.249   |

## Function
Host (10.199.0.3) -> VPN SERVER, DHCP SERVER (192.168.99.0/24)
Host (10.200.0.3) -> VPN BRIDGE

## Routing

### VPN Client (dnsmasq)
```
# Static routing
dhcp-option=option:classless-static-route,10.199.0.0/24,192.168.99.254,10.200.0.0/24,192.168.99.254,10.198.0.0/24,192.168.99.254

```

### VietNam Production <-> VPN Client
```
[hungnp@prod-06.vn:~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp6s0f0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether e8:9a:8f:f1:d0:28 brd ff:ff:ff:ff:ff:ff
    inet X.X.X.X/24 brd 222.255.237.255 scope global noprefixroute enp6s0f0
       valid_lft forever preferred_lft forever
    inet6 fe80::ea9a:8fff:fef1:d028/64 scope link 
       valid_lft forever preferred_lft forever
3: enp6s0f1: <BROADCAST,MULTICAST,PROMISC,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether e8:9a:8f:f1:d0:29 brd ff:ff:ff:ff:ff:ff
    inet 10.199.0.6/24 brd 10.199.0.255 scope global noprefixroute enp6s0f1
       valid_lft forever preferred_lft forever
    inet6 fe80::ea9a:8fff:fef1:d029/64 scope link 
       valid_lft forever preferred_lft forever

[hungnp@prod-06.vn:~]$ sudo ip route add 192.168.99.0/24 via 10.199.0.249 dev enp6s0f1

[hungnp@prod-06.vn:~]$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         X.X.X.X         0.0.0.0         UG    100    0        0 enp6s0f0
10.199.0.0      0.0.0.0         255.255.255.0   U     101    0        0 enp6s0f1
X.X.X.X         0.0.0.0         255.255.255.255 UH    100    0        0 enp6s0f0
192.168.99.0    10.199.0.249    255.255.255.0   UG    0      0        0 enp6s0f1
Z.Z.Z.Z         0.0.0.0         255.255.255.0   U     100    0        0 enp6s0f0

```

### Singapore Production <-> VPN Client
```
[hungnp@prod-02.sg:~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP qlen 1000
    link/ether 00:1e:67:d3:e7:3f brd ff:ff:ff:ff:ff:ff
    inet X.X.X.X/26 brd 209.58.162.63 scope global eth0
    inet6 fe80::21e:67ff:fed3:e73f/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP qlen 1000
    link/ether 00:1e:67:d3:e7:40 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::21e:67ff:fed3:e740/64 scope link 
       valid_lft forever preferred_lft forever
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP qlen 1000
    link/ether 00:1e:67:d3:e7:41 brd ff:ff:ff:ff:ff:ff
    inet 10.200.0.2/24 brd 10.200.0.255 scope global eth2
    inet6 fe80::21e:67ff:fed3:e741/64 scope link 
       valid_lft forever preferred_lft forever
5: eth3: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN qlen 1000
    link/ether 00:1e:67:d3:e7:42 brd ff:ff:ff:ff:ff:ff

[hungnp@prod-02.sg:~]$ sudo ip route add 192.168.99.0/24 via 10.200.0.249 dev eth2 

[hungnp@prod-02.sg:~]$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
z.z.z.z         0.0.0.0         255.255.255.192 U     0      0        0 eth0
192.168.99.0    10.200.0.249    255.255.255.0   UG    0      0        0 eth2
10.200.0.0      0.0.0.0         255.255.255.0   U     0      0        0 eth2
192.168.10.0    10.200.0.254    255.255.255.0   UG    0      0        0 eth2
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     1003   0        0 eth1
169.254.0.0     0.0.0.0         255.255.0.0     U     1004   0        0 eth2
0.0.0.0         x.x.x.x  0.0.0.0         UG    0      0        0 eth0

```

## VPN CLIENT ACCESS
```
VPN Client>AccountList
AccountList command - Get List of VPN Connection Settings
Item                        |Value
----------------------------+---------------------------------------------
VPN Connection Setting Name |hungnp
Status                      |Connected
VPN Server Hostname         |VPN_SERVER_IP:PORT (Direct TCP/IP Connection)
Virtual Hub                 |ANTS_LOCAL_BRIDGE
Virtual Network Adapter Name|vpn
The command completed successfully.

# Request DHCP

root@epson 10:16:24 ~ 
$ dhclient -v vpn_vpn
Internet Systems Consortium DHCP Client 4.4.1
Copyright 2004-2018 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/

Listening on LPF/vpn_vpn/5e:33:e4:9f:4b:43
Sending on   LPF/vpn_vpn/5e:33:e4:9f:4b:43
Sending on   Socket/fallback
DHCPREQUEST for 192.168.99.11 on vpn_vpn to 255.255.255.255 port 67
DHCPACK of 192.168.99.11 from 192.168.99.1
bound to 192.168.99.11 -- renewal in 20601 seconds.

root@epson 10:16:27 ~ 
$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.1.1     0.0.0.0         UG    600    0        0 wlan0
10.110.0.0      0.0.0.0         255.255.255.0   U     0      0        0 vmnet10
10.111.0.0      0.0.0.0         255.255.255.0   U     0      0        0 vmnet11
10.112.0.0      0.0.0.0         255.255.255.0   U     0      0        0 vmnet12
10.113.0.0      0.0.0.0         255.255.255.0   U     0      0        0 vmnet13
10.114.0.0      0.0.0.0         255.255.255.0   U     0      0        0 vmnet14
10.198.0.0      192.168.99.254  255.255.255.0   UG    0      0        0 vpn_vpn
10.199.0.0      192.168.99.254  255.255.255.0   UG    0      0        0 vpn_vpn
10.200.0.0      192.168.99.254  255.255.255.0   UG    0      0        0 vpn_vpn
172.16.1.0      0.0.0.0         255.255.255.0   U     0      0        0 vmnet8
192.168.0.0     0.0.0.0         255.255.252.0   U     600    0        0 wlan0
192.168.32.0    0.0.0.0         255.255.255.0   U     0      0        0 eth0
192.168.99.0    0.0.0.0         255.255.255.0   U     0      0        0 vpn_vpn


```


