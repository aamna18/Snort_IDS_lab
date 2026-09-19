# Test Results

| Test | Result |
|---|---|
| IPFire connectivity | Verified |
| Kali → Metasploitable routing | Verified |
| TCP reconnaissance | Performed |
| UDP reconnaissance | Performed |
| Nikto testing | Performed |
| VSFTPD testing | Performed |
| Snort passive capture | Verified |
| Snort configuration validation | Successful |
| JSON alert logging | Verified |
| Snort evidence | 1,358 records |

## Key Finding

Snort successfully observed and logged TCP SYN activity generated during reconnaissance.

The custom rule provided packet-level detection but did not aggregate the traffic into a dedicated port-scan event.

IPFire provided the separate firewall enforcement and firewall logging layer.
