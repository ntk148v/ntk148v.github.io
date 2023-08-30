---
title: "Bgp Ecmp Load Balancing"
date: 2022-02-07T15:35:08+07:00
draft: false
comments: true
tags: ["tech", "network", "bgp", "ecmp"]
---

## Introduction

We will build a Load balancer with [BGP](https://en.wikipedia.org/wiki/Border_Gateway_Protocol) and [Equal-Cost Multipath routing (ECMP)](https://en.wikipedia.org/wiki/Equal-cost_multi-path_routing) using both [Bird](https://bird.network.cz/) and [ExaBGP](https://github.com/Exa-Networks/exabgp).

References:

- [How to build a load balancer with BGP and ECMP using VyOS](https://gist.github.com/bufadu/0c3ba661c141a2176cd048f65430ae8d)
- [Multi-tier load balancer](https://vincent.bernat.ch/en/blog/2018-multi-tier-loadbalancer#first-tier-ecmp-routing)
- [Load balancing without Load balancers](https://blog.cloudflare.com/cloudflares-architecture-eliminating-single-p/)

## Lab overview

- [EVE-NG](https://www.eve-ng.net/) version 2.0.3-112
- QEMU version 2.4.0

{{< figure class="figure" src="/photos/bgp-ecmp-load-balancing/ecmp-bgp.png" >}}

- `AS 65000`: internet service provider. In this post, we will build a BGP session between EdgeRouter and ISP router.
- `ISPRouter` and `EdgeRouter` are [Fortinet Fortigate v7.0.3](https://docs.fortinet.com/document/fortigate/7.0.3) instances. You can use other routers as well.
- `Switch` is a Cisco switch.
- `client`, `lb1`, and `lb2` are Ubuntu server 18.04 instances. `lb1` and `lb2` will be in the `10.12.12.0/24` private LAN, we will install nginx (LB L7) on these. Both servers will announce the same public IP (10.13.13.1) to `EdgeRouter` using BGP. Incoming traffic from internet to this public IP will be routed to `lb1` or `lb2` depending of a hash.
- You need to download and install device virtual images. Follow [EVE-NG guide](https://www.eve-ng.net/index.php/documentation/howtos/howto-add-fortinet-images/).

## Configure

###  ISPRouter

Follow the [Fortigate document](https://docs.fortinet.com/document/fortigate/7.0.3) for the basic commands.

```bash
# interfaces configurations
config system interface
    edit "port1"
        set mode static
        set ip 192.168.22.226 255.255.255.0
        set allowaccess ping https ssh http telnet
    next
    edit "port2"
        set mode static
        set ip 10.0.0.254 255.255.255.0
        set allowaccess ping https ssh http telnet
    next
    edit "port3"
        set mode static
        set ip 172.16.42.2 255.255.255.254
        set allowaccess ping https ssh http telnet
    next
end
# Simple BGP configuration
config router bgp
    set as 65000
    set router-id 172.16.42.2
    config neighbor
        edit "172.16.42.3"
            set capability-default-originate enable
            set remote-as 65500
            set update-source "port3"
        next
    end
end

# Firewall policy (simple allow all)
config firewall policy
    edit 1
        set name "allow"
        set srcintf "any"
        set dstintf "any"
        set action accept
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service "ALL"
    next
end
```

###  EdgeRouter

```bash
config system interface
    edit "port1"
        set vdom "root"
        set ip 172.16.42.3 255.255.255.254
        set allowaccess ping https ssh snmp http telnet
        set type physical
        set snmp-index 1
    next
    edit "port2"
        set vdom "root"
        set ip 10.12.12.254 255.255.255.0
        set allowaccess ping https ssh snmp http telnet
        set type physical
        set snmp-index 2
    next
end

# BGP config
config router bgp
    set as 65500
    set router-id 172.16.42.3
    set ibgp-multipath enable
    set additional-path enable
    config neighbor
        edit "10.12.12.2"
            set remote-as 65500
            set update-source "port2"
        next
        edit "10.12.12.1"
            set remote-as 65500
            set update-source "port2"
        next
        edit "172.16.42.2"
            set remote-as 65000
            set update-source "port1"
        next
    end
end

# Firewall policy (simple allow all)
config firewall policy
    edit 1
        set name "allow"
        set srcintf "any"
        set dstintf "any"
        set action accept
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service "ALL"
    next
end
```

- You can change [load-balancing algorithms](https://docs.fortinet.com/document/fortigate/7.0.3/administration-guide/25967/equal-cost-multi-path). By default, it is `source-ip-based`.

```bash
config system settings
    set v4-ecmp-mode {source-ip-based* | weight-based | usage-based | source-dest-ip-based}
end
```

###  Switch

- Set up mode access.

```bash
configure terminal
interface range Ethernet 0/0-2
switchport mode access
switchport access  vlan 2
exit
copy running-config start-config
```

###  Servers

- Configure `10.13.13.1` on the local loopback interface.

```bash
lb1$ sudo cat <<EOF > /etc/netplan/00-installer-config.yaml
network:
  ethernets:
    lo:
      addresses:
      - 10.13.13.1/24
    ens3:
      addresses:
      - 10.12.12.1/24 # change to 10.12.12.2 on lb2
      gateway4: 10.12.12.254
  version: 2
EOF
```

```bash
lb1$ sudo netplan apply
```

- Install nginx.

```bash
lb1$ sudo apt install nginx -y
```

- Configure nginx:

```bash
lb1$ sudo cat <<EOF > /etc/nginx/sites-enabled/default
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;
}
EOF

# In lb1
lb1$ echo lb1 > /var/www/html/hostname
# In lb2
lb2$ echo lb2 > /var/www/html/hostname

lb1$ sudo service nginx start
```

Choose one of the follow configurations (Bird or ExaBGP).

#### . Configure Bird

- We will use [Bird](https://github.com/CZ-NIC/bird).
- Install bird

```bash
lb1$ sudo apt install  bird -y
```

- Configure bird.

```bash
lb1$ sudo cat <<EOF > /etc/bird.conf
protocol kernel {
        persist;
        scan time 20;
        export all;
}

protocol device {
        scan time 10;
}

protocol static {
}

protocol static static_bgp {
        import all;
        route 10.13.13.1/32 reject;
}

protocol bgp {
        local as 65500;
        neighbor 10.12.12.254 as 65500;
        import none;
        export where proto: "static_bgp";
}
EOF
```

- Start bird.

```bash
lb1$ sudo service bird start
lb1$ sudo service bird status
_ bird.service - BIRD Internet Routing Daemon (IPv4)
   Loaded: loaded (/lib/systemd/system/bird.service; disabled; vendor preset: enabled)
   Active: inactive (dead)

Feb 07 08:11:13 lb2 systemd[1]: Starting BIRD Internet Routing Daemon (IPv4)...
Feb 07 08:11:13 lb2 systemd[1]: Started BIRD Internet Routing Daemon (IPv4).
Feb 07 08:11:13 lb2 bird[884]: Chosen router ID 10.12.12.2 according to interface ens3
Feb 07 08:11:13 lb2 bird[884]: Started
```

#### . Configure ExaBGP

- Instead of Bird, we can also use [ExaBGP](https://github.com/Exa-Networks/exabgp). ExaBGP provides a convenient way to implement Software Defined Networking by transforming BGP messages into friendly plain text or JSON, which can then be easily handled by simple scripts or your BSS/OSS.
- Install ExaBGP

```bash
lb1$ sudo apt install python3-pip python3-dev -y
lb1$ sudo pip3 install exabgp
lb1$ exabgp --run healthcheck --help
```

- Create `exabgp` user and group.

```bash
lb1$ sudo useradd exabgp
```

- Configure ExaBGP

```bash
lb1$ sudo cat <<EOF > /etc/exabgp/exabgp.conf
neighbor 10.12.12.254 {  # Remote neighbor to peer with
    router-id 10.12.12.1; # Local router-id, change to 10.12.12.2 on lb2
    local-address 10.12.12.1; # Local update-router, change to 10.12.12.2 on lb2
    local-as 65500; # Local AS
    peer-as 65500; # Peer AS

    family {
        ipv4 unicast;
    }
}

process watch-nginx {
    run python3 -m exabgp healthcheck --cmd "curl -sf http://10.13.13.1" --label nginx --ip 10.13.13.1/32;
    encoder json;
}
EOF
```

- Configure ExaBGP service

```bash
lb1$ sudo cat <<EOF > /lib/systemd/system/exabgp.service
[Unit]
Description=ExaBGP
Documentation=man:exabgp(1)
Documentation=man:exabgp.conf(5)
Documentation=https://github.com/Exa-Networks/exabgp/wiki
After=network.target
ConditionPathExists=/etc/exabgp/exabgp.conf

[Service]
User=exabgp
Group=exabgp
Environment=exabgp_daemon_daemonize=false
RuntimeDirectory=exabgp
RuntimeDirectoryMode=0750
ExecStart=/usr/local/bin/exabgp /etc/exabgp/exabgp.conf
ExecReload=/bin/kill -USR1 $MAINPID
Restart=always
CapabilityBoundingSet=CAP_NET_ADMIN
AmbientCapabilities=CAP_NET_ADMIN

[Install]
WantedBy=multi-user.target
EOF

lb1$ sudo service exabgp start
lb1$ sudo service exabgp status
```

## Validate

To make sure everything works as expected.

###  Scenario 1: Both servers are OK

- Client check.

```bash
client$ curl http://10.13.13.1/hostname
lb1
```

-

- Change client ip. The load balancing works!

```bash
# Set client's ip to 10.0.0.1
client$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:50:00:00:05:00 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.1/24 brd 10.0.0.255 scope global ens3
       valid_lft forever preferred_lft forever
    inet6 fe80::250:ff:fe00:500/64 scope link
       valid_lft forever preferred_lft forever

client$ curl http://10.13.13.1/hostname
lb1
```

- Change client's ip address to `10.0.0.100` and send another request. You can the request was sent to `lb2` instead `lb1`.

```bash
client$ curl http://10.13.13.1/hostname
lb2
```

- Check EdgeRouter.

```
FortiGate-VM64-KVM # get router info routing-table all
Codes: K - kernel, C - connected, S - static, R - RIP, B - BGP
       O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default

Routing table for VRF=0
B*      0.0.0.0/0 [20/0] via 172.16.42.2 (recursive is directly connected, port1), 01:01:52
C       10.12.12.0/24 is directly connected, port2
B       10.13.13.1/32 [200/100] via 10.12.12.1 (recursive is directly connected, port2), 01:04:33
                      [200/100] via 10.12.12.2 (recursive is directly connected, port2), 01:04:33
C       172.16.42.2/31 is directly connected, port1

FortiGate-VM64-KVM # get router info bgp network
VRF 0 BGP table version is 3, local router ID is 172.16.42.3
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight RouteTag Path
*> 0.0.0.0/0        172.16.42.2              0             0        0 65000 i <-/1>
*>i10.13.13.1/32    10.12.12.2             100    100      0        0 i <-/2>
*>i                 10.12.12.1             100    100      0        0 i <-/1>

Total number of prefixes 2
```

- Check ISPRouter

```
FortiGate-VM64-KVM # get router info routing-table all
Codes: K - kernel, C - connected, S - static, R - RIP, B - BGP
       O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default

Routing table for VRF=0
C       10.0.0.0/24 is directly connected, port2
B       10.13.13.1/32 [20/0] via 172.16.42.3 (recursive is directly connected, port3), 01:06:48
C       172.16.42.2/31 is directly connected, port3
C       192.168.22.0/24 is directly connected, port1


FortiGate-VM64-KVM # get router info bgp network
VRF 0 BGP table version is 1, local router ID is 172.16.42.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight RouteTag Path
*> 10.13.13.1/32    172.16.42.3              0             0        0 65500 i <-/1>

Total number of prefixes 1
```

###  Scenario 2: One lb is down

- Stop nginx on lb2 (ExaBGP only, Bird may require the complete shutdown) or stop lb2 physically.

```bash
lb2$ sudo service nginx stop
lb2$ sudo journalctl -fu exabgp
-- Logs begin at Thu 2022-01-27 12:07:29 UTC. --
Feb 07 11:12:11 lb2 healthcheck[13584]: send announces for DOWN state to ExaBGP
Feb 07 11:12:11 lb2 exabgp[13562]: api             route added to neighbor 10.12.12.254 local-ip 10.12.12.2 local-as 65500 peer-as 65500 router-id 10.12.12.254 family-allowed in-open : 10.13.13.1/32 next-hop self med 1000
```

- Check client

```bash
client$ curl 10.13.13.1/hostname
lb1
```

- Check EdgeRouter, you may notice that the metric of 10.12.12.2 hop became 1000.

```bash
FortiGate-VM64-KVM # get router info bgp network
VRF 0 BGP table version is 4, local router ID is 172.16.42.3
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight RouteTag Path
*> 0.0.0.0/0        172.16.42.2              0             0        0 65000 i <-/1>
* i10.13.13.1/32    10.12.12.2            1000    100      0        0 i <-/->
*>i                 10.12.12.1             100    100      0        0 i <-/1>

Total number of prefixes 2
```

- Bring it back, then check client and EdgeRouter.

```
FortiGate-VM64-KVM # get router info bgp network
VRF 0 BGP table version is 5, local router ID is 172.16.42.3
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight RouteTag Path
*> 0.0.0.0/0        172.16.42.2              0             0        0 65000 i <-/1>
*>i10.13.13.1/32    10.12.12.2             100    100      0        0 i <-/2>
*>i                 10.12.12.1             100    100      0        0 i <-/1>

Total number of prefixes 2
```

- Check client again.

```bash
client$ curl 10.13.13.1/hostname
lb2
```
