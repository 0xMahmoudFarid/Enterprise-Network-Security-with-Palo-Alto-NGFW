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

![Rule]

<img width="1686" height="649" alt="2" src="https://github.com/user-attachments/assets/205aa249-df91-4d4e-82d2-26b164a23df8" />


## Verification
- curl from Kali to the DMZ web server returns HTTP 200. The traffic log identifies it as web-browsing and matches Allow-Trust-to-DMZ.
- ping between the same two hosts is now blocked. The log shows it hitting Deny-All-Interzone.
- Same source, same destination: the firewall allowed or denied based on the application, not just the IP.

![curl]

<img width="1473" height="888" alt="3" src="https://github.com/user-attachments/assets/03eeee48-4d51-423a-b03f-9c10b69f6d91" />

![ping blocked]

<img width="1473" height="888" alt="3" src="https://github.com/user-attachments/assets/c7e78728-fc3a-456b-a4cc-c978b2e54735" />

![Allow log]

<img width="1674" height="380" alt="1" src="https://github.com/user-attachments/assets/7a27f231-a7c3-4012-8188-171317af8de4" />

![Deny log]

<img width="1674" height="380" alt="1" src="https://github.com/user-attachments/assets/297f241c-1969-41cc-8783-ef8e5f9a9d9c" />


## Notes
- The Phase 2 screenshots show the earlier, broader version of the Trust-to-DMZ rule.
- This lab has no Palo Alto subscriptions. App-ID works with the content bundled in the image, with no content updates. Threat Prevention, URL Filtering, and WildFire are not part of this lab.

## Next ideas
- Test an application on a non-default port.
- SSL decryption and zone protection.
