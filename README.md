# GNS3 Network Security Lab -- IPFire Firewall & Snort 3 IDS

## 1. Project Overview

This project implements a segmented network-security laboratory using
GNS3, IPFire, Snort 3, Kali Linux, Metasploitable 2, and Ubuntu.

The laboratory demonstrates:

-   Network segmentation and firewall enforcement
-   Controlled reconnaissance from Kali Linux
-   Monitoring of internal network traffic
-   Passive IDS operation with Snort 3
-   Structured JSON alert logging
-   Testing against an intentionally vulnerable host
-   Comparison of IDS visibility with firewall enforcement
-   Collection of evidence for security-event analysis

> **Lab environment:** All testing described in this repository was
> performed in an isolated laboratory environment using intentionally
> vulnerable systems.

------------------------------------------------------------------------

## 2. Architecture

``` text
                         External / Untrusted Network
                              192.168.100.0/24
                                      |
                                      |
                              +---------------+
                              |     Kali      |
                              | 192.168.100.21|
                              +-------+-------+
                                      |
                                  Cloud1
                                      |
                              +-------v-------+
                              |    IPFire     |
                              |               |
                              | RED:  .100.62 |
                              | GREEN: .10.1   |
                              +-------+-------+
                                      |
                                192.168.10.0/24
                                      |
                                    Hub1
                         _____________|_____________
                        /             |             \
                       /              |              \
                  Cloud2          Snort-Logger       |
                    |             192.168.10.104     |
                 VMnet2                                |
              ____________                             |
             /            \                            |
            /              \                           |
   Metasploitable 2     Ubuntu Secure                  |
    .100                .102                          |
```

### Network zones

  Component          Network / IP      Purpose
  ------------------ ----------------- ---------------------------------
  Kali Linux         192.168.100.21    External attacker/test machine
  IPFire RED         192.168.100.62    External/untrusted interface
  IPFire GREEN       192.168.10.1      Internal gateway
  Metasploitable 2   192.168.10.100    Intentionally vulnerable target
  Ubuntu Secure      192.168.10.102    Secure/admin test host
  Snort-Logger       192.168.10.104    Passive IDS and log collector
  GREEN network      192.168.10.0/24   Protected internal LAN

IPFire DHCP was configured for the GREEN network using the
192.168.10.100--192.168.10.200 range.

------------------------------------------------------------------------

## 3. Technologies Used

-   **GNS3** -- Network topology and virtual networking
-   **IPFire 2.29 Core 203** -- Firewall/router
-   **Snort 3.12.2.0** -- Passive network IDS
-   **Kali Linux** -- Security testing and reconnaissance
-   **Metasploitable 2** -- Deliberately vulnerable target
-   **Ubuntu Server** -- Snort monitoring/logging system
-   **Ubuntu Desktop** -- Secure/admin test host
-   **VMware** -- Virtual machine platform
-   **AFPacket DAQ** -- Linux live packet acquisition

------------------------------------------------------------------------

## 4. IPFire Configuration

IPFire separates the external and internal networks:

-   **RED:** 192.168.100.0/24
-   **GREEN:** 192.168.10.0/24
-   **GREEN gateway:** 192.168.10.1

The firewall was configured to permit controlled laboratory traffic from
Kali to the Metasploitable target while retaining firewall logging.

A laboratory rule was created for:

-   Source: `192.168.100.21`
-   Destination: `192.168.10.100`
-   Protocol: All
-   Action: ACCEPT
-   Logging: Enabled

This rule allowed controlled testing against the vulnerable host while
IPFire continued to record forwarding activity.

------------------------------------------------------------------------

## 5. Snort Deployment

Snort 3 was compiled and installed on Ubuntu Server.

The installation included:

1.  LibDAQ 3
2.  Snort 3.12.2.0
3.  AFPacket DAQ
4.  Custom local detection rules
5.  JSON alert logging

Snort was operated in **passive mode** on interface `ens3`.

The configuration was successfully validated with zero warnings.

### Custom laboratory rule

``` text
alert tcp any any -> 192.168.10.0/24 any (flags:S; msg:"LAB TCP SYN activity"; sid:1000001; rev:1;)
```

The rule was intentionally simple and was used to demonstrate TCP SYN
visibility and structured alert generation.

------------------------------------------------------------------------

## 6. Monitoring Architecture

Snort was connected to the internal network through an Ethernet hub.

This allowed Snort to observe traffic without becoming an inline
dependency.

``` text
Kali
  |
IPFire
  |
GREEN Network
  |
 Hub
  |
  +---- Metasploitable
  |
  +---- Ubuntu Secure
  |
  +---- Snort
```

The design separates:

-   **Firewall enforcement** -- IPFire
-   **Traffic observation/detection** -- Snort
-   **Vulnerable target** -- Metasploitable
-   **Security testing** -- Kali

------------------------------------------------------------------------

## 7. Security Testing Performed

The following controlled activities were performed during the laboratory
session.

### 7.1 TCP reconnaissance

Kali performed TCP reconnaissance against the internal network and
Metasploitable.

The Snort master dataset recorded individual TCP SYN packets generated
by the reconnaissance activity.

### 7.2 Service enumeration

Metasploitable exposed multiple intentionally vulnerable services,
including:

-   SSH
-   Telnet
-   SMTP
-   DNS
-   HTTP
-   NetBIOS/SMB
-   MySQL
-   PostgreSQL
-   FTP

### 7.3 Web reconnaissance

Nikto was used against the vulnerable environment as part of the
reconnaissance phase.

### 7.4 UDP reconnaissance

UDP-based reconnaissance was also performed to generate additional
network-security telemetry.

### 7.5 VSFTPD laboratory activity

FTP/VSFTPD testing was performed against the intentionally vulnerable
Metasploitable host.

The Snort evidence contains traffic associated with TCP/21 and
subsequent TCP/6200 activity.

------------------------------------------------------------------------

## 8. Snort Evidence

The final Snort master dataset contains:

**1,358 JSON records**

Master file:

``` text
/var/log/snort/LAB_MASTER_2026-09-19.json
```

The JSON records contain fields such as:

-   Timestamp
-   Packet number
-   Protocol
-   Packet length
-   Source IP/port
-   Destination IP/port
-   Rule identifier
-   Action

Example structure:

``` json
{
  "timestamp": "...",
  "proto": "TCP",
  "src_ap": "192.168.100.21:xxxxx",
  "rule": "1:1000001:1",
  "action": "allow"
}
```

------------------------------------------------------------------------

## 9. Important Detection Finding

The TCP reconnaissance scan was **visible to Snort and recorded
packet-by-packet**, but the current custom rule did not aggregate the
sequence into a dedicated "port scan" detection event.

This distinction is important:

-   Snort **observed and logged** the TCP SYN traffic.
-   The custom rule **matched individual SYN packets**.
-   The rule did **not perform higher-level scan aggregation**.
-   Therefore, `action: allow` should not be interpreted as "the traffic
    was invisible."
-   IPFire separately provided firewall enforcement and firewall-event
    logging.

The laboratory therefore demonstrates the difference between:

**packet visibility → rule matching → event classification → firewall
enforcement**

------------------------------------------------------------------------

## 10. Firewall and IDS Correlation

IPFire and Snort provided complementary evidence.

### Snort

Snort provided packet-level visibility of the reconnaissance and
service-testing traffic on the monitored internal network.

### IPFire

IPFire recorded firewall forwarding and input decisions, including:

-   `FORWARDFW`
-   `DROP_FORWARD`
-   `DROP_INPUT`
-   Connection-tracking related events

Later firewall activity from Kali included attempts involving internal
hosts and the IPFire WebGUI at:

``` text
192.168.10.1:444
```

The firewall therefore provided an enforcement layer independent of
Snort's packet logging.

------------------------------------------------------------------------

## 11. Background Traffic

Not every firewall event should be considered malicious.

The firewall logs also contained background LAN traffic involving hosts
such as:

-   192.168.100.11
-   192.168.100.27
-   192.168.100.8

Examples included NetBIOS and local-discovery traffic.

There were also connection-tracking events associated with routine
Ubuntu/Canonical update traffic.

These events demonstrate why security logs require contextual analysis.

------------------------------------------------------------------------

## 12. Evidence Files

The principal evidence produced by the laboratory includes:

``` text
LAB_MASTER_2026-09-19.json
```

This contains the final Snort JSON dataset with 1,358 records.

The project report documents:

-   Network architecture
-   IP addressing
-   IPFire configuration
-   Snort installation
-   Snort rule configuration
-   Reconnaissance testing
-   VSFTPD testing
-   Logging methodology
-   Security findings
-   Limitations and future improvements

------------------------------------------------------------------------

## 13. Repository Structure

Recommended repository structure:

``` text
gns3-network-security-lab/
│
├── README.md
│
├── docs/
│   ├── GNS3_Snort_IDS_Lab_Report_FINAL.docx
│   ├── architecture.md
│   └── attack-detection-workflow.md
│
├── topology/
│   ├── final-topology.png
│   └── gns3-project.gns3
│
├── ip-addressing/
│   └── ip-plan.md
│
├── ipfire/
│   ├── configuration-notes.md
│   └── firewall-rules.md
│
├── snort/
│   ├── snort.lua
│   ├── local.rules
│   └── README.md
│
├── logging/
│   └── README.md
│
├── testing/
│   ├── reconnaissance.md
│   ├── nikto.md
│   ├── udp-recon.md
│   ├── vsftpd-test.md
│   └── results.md
│
└── screenshots/
    ├── topology.png
    ├── ipfire.png
    ├── snort-running.png
    ├── nmap-results.png
    └── snort-alerts.png
```

------------------------------------------------------------------------

## 14. Security and Repository Hygiene

The following should **not** be committed to a public GitHub repository:

-   VMware `.vmdk` files
-   QCOW2/VM disk images
-   ISO images
-   Passwords
-   API keys
-   SSH private keys
-   Real credentials
-   Sensitive personal information
-   Unnecessary system-specific configuration
-   Unredacted logs containing sensitive information

Large raw log datasets should preferably remain as local evidence unless
the repository is private or the logs have been sanitized.

------------------------------------------------------------------------

## 15. Limitations

This project is a laboratory implementation rather than a production IDS
deployment.

Important limitations include:

1.  The custom Snort rule is intentionally simple.
2.  Individual TCP SYN packets are detected, but port-scan aggregation
    is not implemented by the custom rule.
3.  The Snort sensor is passive and therefore does not block traffic.
4.  IPFire provides the enforcement function.
5.  Firewall and Snort timestamps were not always directly aligned
    across the retained evidence files.
6.  A production deployment would normally use a maintained Snort rule
    set and centralized log-management/SIEM infrastructure.
7.  NTP/time synchronization should be enabled consistently across all
    security devices for stronger event correlation.

------------------------------------------------------------------------

## 16. Future Improvements

Potential future enhancements include:

-   Add maintained Snort detection rules for reconnaissance and
    scanning.
-   Configure thresholding and scan-event correlation.
-   Centralize IPFire, Snort and endpoint logs.
-   Deploy a SIEM/log-management platform.
-   Synchronize all systems using NTP.
-   Add packet-capture retention for forensic investigation.
-   Add dashboards for alerts and traffic statistics.
-   Add automated alert classification.
-   Expand the lab with additional VLANs and protected services.

------------------------------------------------------------------------

## 17. Conclusion

The laboratory successfully demonstrated a segmented network-security
architecture using IPFire and passive Snort 3.

Kali generated controlled reconnaissance and service-testing traffic.
IPFire provided network segmentation and firewall enforcement, while
Snort observed internal traffic and produced structured JSON evidence.

The final Snort dataset contains 1,358 records and provides evidence of
reconnaissance, service enumeration, internal traffic and VSFTPD-related
testing.

The most important technical finding is that the current Snort rule
provided packet-level visibility but did not aggregate the observed SYN
sequence into a dedicated port-scan event. This provides a clear
baseline for future improvement through more advanced Snort rules and
centralized security-event correlation.

------------------------------------------------------------------------

## 18. Lab Disclaimer

This project was conducted in a controlled laboratory environment using
intentionally vulnerable systems. The techniques and tests documented
here are intended for authorized security testing, learning, detection
engineering and defensive network-security research.
