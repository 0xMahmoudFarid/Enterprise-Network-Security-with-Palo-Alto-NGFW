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

![Certificates](screenshots/01-certificates.png)
![Decryption rule](screenshots/02-decryption-rule.png)

### Verification
- Before: the traffic log shows the session as ssl with Decrypted = no.
- After: the same traffic shows Decrypted = yes, and the application is identified as web-browsing, because the firewall can now see inside the HTTPS session.
- Certificate check: curl against the server directly shows the original Apache certificate. Through the firewall, the issuer is Lab Forward Untrust CA. The firewall used its Forward Untrust certificate because the server's certificate is self-signed and not trusted.

![Before](screenshots/03-traffic-before-decrypt.png)
![Original cert](screenshots/04-curl-original-cert.png)
![Cert through firewall](screenshots/05-curl-through-firewall.png)
![After](screenshots/06-traffic-after-decrypt.png)

## Part 2: Zone Protection (Reconnaissance Protection)
### What I configured
- Zone Protection profile Recon-Protection with TCP Port Scan and Host Sweep detection: 20 events within 2 seconds, action block-ip, tracked by source, 60 second duration.
- Applied the profile to the Trust zone, where the scan enters the firewall.

![Profile](screenshots/07-zone-protection-profile.png)
![Zone](screenshots/08-zone-trust.png)

### Verification
- Ran an Nmap SYN scan from Kali against the DMZ server.
- The Threat log recorded a TCP Port Scan from 10.0.1.10, and the source was blocked. A curl right after the scan timed out.
- After the block duration expired, curl returned HTTP 200 again.

![Nmap then blocked](screenshots/09-nmap-then-blocked.png)
![Threat log](screenshots/10-threat-log-scan.png)

## Notes and limitations
- At first, curl to the HTTPS port returned a "wrong version number" error. Enabling the default-ssl Apache site and reloading Apache fixed it.
- [Add the problem you fixed during the zone protection test.]
- The firewall's CA certificates are not installed on the client, so curl -k was used and decryption was verified through logs and the certificate issuer.
- The lab has no Palo Alto subscriptions. SSL decryption and Zone Protection do not need one. Threat Prevention, URL Filtering, and WildFire are not part of this lab.