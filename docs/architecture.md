# Lab Architecture

## Overview

The laboratory uses IPFire as the boundary firewall/router and Snort 3 as a passive IDS on the protected internal segment.

## Logical Topology

```text
Kali Linux
192.168.100.21
      |
    Cloud1
      |
IPFire Firewall
RED   192.168.100.62
GREEN 192.168.10.1
      |
     Hub1
  _____|____________________
 /          |              \
Cloud2    Snort IDS       Internal Hosts
 |        192.168.10.104      |
VMnet2                     /    \
                    Metasploitable  Ubuntu Secure
                    192.168.10.100 192.168.10.102
```
Addressing
Component	Address	Role
Kali Linux	192.168.100.21/24	Security testing
IPFire RED	192.168.100.62/24	External interface
IPFire GREEN	192.168.10.1/24	Internal gateway
Metasploitable 2	192.168.10.100/24	Vulnerable target
Ubuntu Secure	192.168.10.102/24	Secure/admin host
Snort-Logger	192.168.10.104/24	Passive IDS
Monitoring Design

Snort is connected to the internal network through an Ethernet hub.

This keeps Snort passive: it observes traffic without being placed inline or becoming responsible for forwarding packets.

IPFire provides firewall enforcement, while Snort provides traffic visibility and alert logging.
