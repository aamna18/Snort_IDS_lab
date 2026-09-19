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
