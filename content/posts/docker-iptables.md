---
title: "Docker and Iptables: You may do it wrong!"
date: 2022-10-25T10:29:11+07:00
tags: ["tech", "docker", "iptables"]
comments: true
---

## Mission

If you're running Docker on a host that is exposed to the Internet (network bridge), you will probably want to restrict external access.

## Docker network

Let's start with a fact that Docker manipulates `iptables` rules to provide network isolation, on Linux. Docker installs custom iptables chains named `DOCKER`, `DOCKER-USER` and `DOCKER-ISOLATION-STAGE-*`, and it ensures that incoming packets are always checked by these chains first.

```shell
iptables -L -n
```

Re.Docker network, I won't describe here because it's a lot of knowledge. You may want to check [Docker network note](https://github.com/ntk148v/til/blob/master/docker/networking/README.md). I will only show iptables packet flow: For example, we have a host with ip 10.0.10.26, then start a container that is exposed port 8000 to internet.

{{< figure class="figure" src="/photos/docker-iptables/docker-iptables-1.png" >}}

This is a basic packet flow from outside:

{{< figure class="figure" src="/photos/docker-iptables/docker-iptables-2.png" >}}

## You may do it wrong - common mistakes

Alright, before we make it right, there are some common mistakes.

### Modify Docker generated rules manually

Docker generates iptables rules, then adds to `DOCKER` chains. Some users may manipulate this chain manually in order to block connections.

_Please don't do it_. Yes, you are able to do it, there is nothing prevent you to perform this kind of action. But every time Docker daemon is reloaded, iptables rules are re-generated and your changes is gone.

{{< quote info >}}
Right: Do not manipulate Docker rules manually.
{{< /quote >}}

### Insert you rules in the wrong chain

iptables basic: iptables is divied into three levels: tables, chains and rules. We only use the filter tables, which contains:

- INPUT: Packets sent to this host pass through this chain.
- OUTPUT: Packets sent from this host pass through this chain.
- FORWARD: Packets forwarded by this host pass through this chain.

Commonly, to block connection from external, put reject rules in INPUT chain. But in Docker, it doesn't work. Look at the above packet flow, the packet doesn't pass through INPUT chain, it goes in FORWARD chain. Since Docker connects the bridge (default docker0) to the default gateway (external interface ens33 for example) via Network Address Translation (NAT) by default, setting INPUT is useless and the FORWARD chain should be set. And since the FORWARD chain defaults to DROP, all forwarding is blocked by DOCKER-USER.

{{< quote info >}}
Right: Add rules which load before Docker's rules, add them to DOCKER-USER.
{{< /quote >}}

### Modify and persistent iptables wrong

You modify and persistent iptables rules like this:

- Save all rules `iptables-save`.
- Modify your rules.
- Restore all rules with `iptables-restore`.
- Persistent and load on start up using iptables-services or rc.local.

This implementation has a drawback, every time Docker container changes, you need to save the current iptables configuration, otherwise when executing `iptables-restore`, it will load the old rules, which will lead to confusing iptables rules.

{{< quote info >}}
Right: Do not save, flush then restore all rules. Check the following solution.
{{< /quote >}}

## Do it right!

### Overview

I have create a repository for this, which is highly inspired by [systemd-service-iptables](https://github.com/boTux-fr/systemd-service-iptables): https://github.com/ntk148v/systemd-iptables

- Whitelist strategy: block all, allow some.
- `iptables-restore -n|--no-flush` turns off implicit global refresh and only performs our manual explicit refresh: only modified chains are flushed.
- Control iptables with systemd: start iptables after other services.

  - iptables.service.

  ```
  [Unit]
  Documentation=man:iptables man:iptables-restore

  [Service]
  Type=oneshot
  ExecStart=/bin/true
  RemainAfterExit=on

  [Install]
  WantedBy=multi-user.target
  ```

  - iptables@.service.

  ```
  [Unit]
  DefaultDependencies=no
  Wants=network-pre.target
  Wants=systemd-modules-load.service
  Wants=local-fs.target
  Before=network-pre.target
  Before=shutdown.target
  After=systemd-modules-load.service
  After=local-fs.target
  After=dbus.service
  After=polkit.service
  PartOf=iptables.service
  Conflicts=shutdown.target
  ConditionPathExists=/etc/iptables/
  Documentation=man:iptables man:iptables-restore

  [Service]
  Environment=IPTABLES_RULES=/etc/iptables/%i.rules
  Environment=IPTABLES_RULES_FLUSH=/etc/iptables/%i.rules.empty

  Type=oneshot
  # Start iptables-restore with no-flush option with %i.rules
  # NoFlush option (-n) is used to not flush other tables than %I.
  ExecStart=/sbin/iptables-restore -v -n $IPTABLES_RULES
  # Reload the rules
  ExecReload=/sbin/iptables-restore -v -n $IPTABLES_RULES
  # Stop and clean with empty rules
  ExecStop=/sbin/iptables-restore -v -n $IPTABLES_RULES_FLUSH
  RemainAfterExit=yes

  [Install]
  WantedBy=multi-user.target
  # Need /etc/iptables/base.rules to exist.
  DefaultInstance=base
  ```

- Templating so nobody can't go wrong:

  - base.rules.empty.

  ```iptables
  *filter

  # Reset counters
  :INPUT ACCEPT [0:0]
  :FORWARD DROP [0:0]
  :OUTPUT ACCEPT [0:0]
  :DOCKER-USER - [0:0]

  # Flush
  -F INPUT
  -F OUTPUT
  -F DOCKER-USER

  COMMIT
  ```

  - base.rules.

  ```iptables
  *filter
  # Reset counters
  :INPUT ACCEPT [0:0]
  :FORWARD DROP [0:0]
  :OUTPUT ACCEPT [0:0]
  :DOCKER-USER - [0:0]

  # Flush
  -F INPUT
  -F OUTPUT
  -F DOCKER-USER

  #########
  # INPUT #
  #########
  # Accept on localhost
  -A INPUT -i lo -m comment --comment "Default: Accept loopback" -j ACCEPT
  # Accept on Docker bridge
  -A INPUT -i br+ -m comment --comment "Default: Accept bridge networks" -j ACCEPT
  -A INPUT -i docker0 -m comment --comment "Default: Accept docker0" -j ACCEPT
  # Allow established sessions to receive traffic
  -A INPUT -m state --state RELATED,ESTABLISHED -m comment --comment "Default: Accept RELATED, ESTABLISHED connections" -j ACCEPT
  -A INPUT -p icmp -m comment --comment "Default: Accept ICMP" -j ACCEPT

  # Insert your INPUT ACCEPT rules here
  # For example:
  # Open https port only for 1 ip:
  # -A INPUT -s 10.1.1.1/32 -p tcp -m tcp --dport 443 -j ACCEPT

  # By default reject all packets and leave a logging
  -A INPUT -m comment --comment "Default: Log INPUT dropped packets" -j LOG --log-prefix "iptables INPUT DROP " --log-level 7
  -A INPUT -j REJECT --reject-with icmp-host-prohibited

  ##########
  # OUTPUT #
  ##########
  # Accept on localhost
  -A OUTPUT -o lo -j ACCEPT
  # Accept Docker networks
  -A OUTPUT -o br+ -m comment --comment "Default: Accept bridge networks" -j ACCEPT
  -A OUTPUT -o docker0 -m comment --comment "Default: Accept docker0" -j ACCEPT
  # Allow established sessions to receive traffic
  -A OUTPUT -m state --state RELATED,ESTABLISHED -m comment --comment "Default: Accept RELATED, ESTABLISHED connections" -j ACCEPT

  # Insert your OUTPUT ACCEPT rules here

  # By default reject all packets and leave a logging
  -A OUTPUT -m comment --comment "Default: Log OUTPUT dropped packets" -j LOG --log-prefix "iptables OUTPUT DROP " --log-level 7
  -A OUTPUT -j REJECT --reject-with icmp-host-prohibited

  ###############
  # DOCKER-USER #
  ###############
  # Change your external interface here!
  # Allow established sessions to receive traffic
  -A DOCKER-USER -i extinf -m state --state established,related -m comment --comment "Default: Accept RELATED, ESTABLISHED connections" -j ACCEPT
  -A DOCKER-USER -i extinf -m conntrack --ctstate RELATED,ESTABLISHED -m comment --comment "Default: Accept RELATED, ESTABLISHED connections" -j ACCEPT
  -A DOCKER-USER -o extinf -m state --state established,related -m comment --comment "Default: Accept RELATED, ESTABLISHED connections" -j ACCEPT
  -A DOCKER-USER -o extinf -m conntrack --ctstate RELATED,ESTABLISHED -m comment --comment "Default: Accept RELATED, ESTABLISHED connections" -j ACCEPT

  # Insert your DOCKER-USER ACCEPT rules here
  # Note that you have to use conntrack and ctorigdstport due to NAT.
  # For example:
  # Allow all on http
  # -A DOCKER-USER -i ens192 -p tcp -m tcp -m conntrack --ctorigdstport 80 -j ACCEPT

  # By default reject all packets
  -A DOCKER-USER -i extinf -m comment --comment "Default: Log DROP-USER input dropped packets in external inteface" -j LOG --log-prefix "iptables DOCKER-USER INPUT DROP in external interface" --log-level 7
  -A DOCKER-USER -i extinf -j REJECT
  -A DOCKER-USER -o extinf -m comment --comment "Default: Log DROP-USER output dropped packets in external inteface" -j LOG --log-prefix "iptables DOCKER-USER OUTPUT DROP in external interface" --log-level 7
  -A DOCKER-USER -o extinf -j REJECT

  -A DOCKER-USER -j RETURN

  ## Commit
  COMMIT
  ```

### Getting started

- Ofc you need iptables and systemd installed.
- On the Linux, run as root:

```shell
git clone https://github.com/ntk148v/systemd-iptables
cd systemd-iptables
# Edit the rules in etc/iptables/base.rules as needed.
# and install the service
cp -Rv etc/. /etc/
```

- Make changes in `/etc/iptables/base.rules`.
  - Replace the placeholder `extinf` interface in the rulebook with your actual external interface (`eth0` for e.x).
  - Add your custom allow rules in the right place!
    - Allow inbound connections -> INPUT Custom Accept Block.
    - Allow outbound connections -> OUTPUT Custom Accept Block.
    - Allow inbound/outbound connections to your Docker containers using network bridge -> DOCKER-USER Custom Accept Block.
- After that, enable the serivces and we are done:

```shell
systemctl daemon-reload
systemctl enable iptables.service
systemctl enable iptables@base.service
systemctl start iptables@base.service
# Check status
systemctl status iptables@base.service
```

- If you make any changes in the future, make sure to restart/reload your service.

```shell
systemctl restart iptables@base.service
```

## References

1. https://docs.docker.com/network/iptables/
2. https://github.com/boTux-fr/systemd-service-iptables
