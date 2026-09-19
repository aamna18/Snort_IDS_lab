# Snort 3 IDS

Snort 3 was deployed on Ubuntu Server as a passive Network Intrusion Detection System (NIDS).

## Configuration

- Interface: `ens3`
- IP Address: `192.168.10.104`
- DAQ: AFPacket
- Mode: Passive
- Log Directory: `/var/log/snort`

## Detection Rule

```text
alert tcp any any -> 192.168.10.0/24 any (flags:S; msg:"LAB TCP SYN activity"; sid:1000001; rev:1;)
