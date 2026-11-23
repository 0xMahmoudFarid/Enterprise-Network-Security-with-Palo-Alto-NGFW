# Phase 4: SSL Decryption and Zone Protection

Adds SSL Forward Proxy decryption for traffic from Trust to the DMZ web server, and Reconnaissance Protection that detects and blocks port scans.

## Environment
- Kali Linux (Trust, 10.0.1.10) as the client and scan source
- Ubuntu with Apache (DMZ, 10.0.2.10), HTTPS enabled with the default self-signed certificate
- Palo Alto VM-Series from the earlier phases, with no subscriptions

## Part 1: SSL Forward Proxy decryption
### What I configured
- Generated two self-signed CA certificates on the firewall: Forward-Trust-CA and Forward-Untrust-CA, marked as Forward Trust and Forward Untrust.
- Added ssl to Allow-Trust-to-DMZ alongside web-browsing.
- Created the decryption rule Decrypt-Trust-to-DMZ (Trust to DMZ, action decrypt, type SSL Forward Proxy).

![Certificates]

<img width="1650" height="238" alt="2" src="https://github.com/user-attachments/assets/31be194f-9af4-4490-b964-d8f945e69834" />

![Decryption rule]

<img width="1711" height="194" alt="3" src="https://github.com/user-attachments/assets/68030653-681f-4a74-9cdc-51b003441fce" />


### Verification
- Before: the traffic log shows the session as ssl with Decrypted = no.
- After: the same traffic shows Decrypted = yes, and the application is identified as web-browsing, because the firewall can now see inside the HTTPS session.
- Certificate check: curl against the server directly shows the original Apache certificate. Through the firewall, the issuer is Lab Forward Untrust CA. The firewall used its Forward Untrust certificate because the server's certificate is self-signed and not trusted.

![Before]

<img width="1672" height="574" alt="4" src="https://github.com/user-attachments/assets/80b26c5b-9c5a-4ed1-ac98-1714a6550a9b" />

![After]

<img width="1672" height="574" alt="4" src="https://github.com/user-attachments/assets/0e7c36ed-ae3f-46d8-ab68-c9240fc49946" />


## Part 2: Zone Protection (Reconnaissance Protection)
### What I configured
- Zone Protection profile Recon-Protection with TCP Port Scan and Host Sweep detection: 20 events within 2 seconds, action block-ip, tracked by source, 60 second duration.
- Applied the profile to the Trust zone, where the scan enters the firewall.

![Profile]

<img width="1668" height="149" alt="6" src="https://github.com/user-attachments/assets/58bcdd0a-8ea7-49b5-ae68-b7d799f70aaf" />


![Zone]

<img width="1657" height="259" alt="5" src="https://github.com/user-attachments/assets/1eebcff0-7004-48f6-947a-977fab5b8791" />


### Verification
- Ran an Nmap SYN scan from Kali against the DMZ server.
- The Threat log recorded a TCP Port Scan from 10.0.1.10, and the source was blocked. A curl right after the scan timed out.
- After the block duration expired, curl returned HTTP 200 again.

![Nmap then blocked]

<img width="1535" height="868" alt="7" src="https://github.com/user-attachments/assets/2f4e23ce-2a3f-4819-b6f3-b23b40db4de9" />

![Threat log]

<img width="1684" height="279" alt="8" src="https://github.com/user-attachments/assets/ac844e74-aaba-4feb-9c5f-a2114c4ff59d" />


## Notes and limitations
- At first, curl to the HTTPS port returned a "wrong version number" error. Enabling the default-ssl Apache site and reloading Apache fixed it.
- [Add the problem you fixed during the zone protection test.]
- The firewall's CA certificates are not installed on the client, so curl -k was used and decryption was verified through logs and the certificate issuer.
- The lab has no Palo Alto subscriptions. SSL decryption and Zone Protection do not need one. Threat Prevention, URL Filtering, and WildFire are not part of this lab.
