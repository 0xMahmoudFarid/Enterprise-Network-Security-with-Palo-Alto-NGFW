# Phase 3: Application-Based Policy (App-ID)

Replaces broad allow rules between zones with an application-based rule, tested with real traffic from a Kali client in Trust to an Apache web server in the DMZ.

## Environment
- Kali Linux (Trust, 10.0.1.10, VMnet2)
- Ubuntu with Apache (DMZ, 10.0.2.10, VMnet3)
- Palo Alto VM-Series from the earlier phases
- These VMs attach directly to the VMware networks, so GNS3 is not used in this phase.

## What I configured
- Narrowed Allow-Trust-to-DMZ from icmp, ping, traceroute, web-browsing, and ssl down to web-browsing only, with the service set to application-default.
- Kept Deny-All-Interzone below it, with logging.

![Rule](screenshots/01-security-rule-web-only.png)

## Verification
- curl from Kali to the DMZ web server returns HTTP 200. The traffic log identifies it as web-browsing and matches Allow-Trust-to-DMZ.
- ping between the same two hosts is now blocked. The log shows it hitting Deny-All-Interzone.
- Same source, same destination: the firewall allowed or denied based on the application, not just the IP.

![curl](screenshots/02-curl-success.png)
![ping blocked](screenshots/03-ping-blocked.png)
![Allow log](screenshots/04-traffic-allow-webbrowsing.png)
![Deny log](screenshots/05-traffic-deny-ping.png)

## Notes
- The Phase 2 screenshots show the earlier, broader version of the Trust-to-DMZ rule.
- This lab has no Palo Alto subscriptions. App-ID works with the content bundled in the image, with no content updates. Threat Prevention, URL Filtering, and WildFire are not part of this lab.

## Next ideas
- Test an application on a non-default port.
- SSL decryption and zone protection.