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
```
The rule detects TCP SYN activity directed toward the internal network.

Validation
sudo snort --daq-dir /usr/local/lib/daq_s3/lib/daq --daq afpacket -i ens3 -c /usr/local/etc/snort/snort.lua -T
Logging

Snort generated JSON alerts in:

/var/log/snort/

The final laboratory evidence contained 1,358 JSON records
