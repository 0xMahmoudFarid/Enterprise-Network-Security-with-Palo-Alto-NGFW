# Phase 2: DMZ and Destination NAT

Extends the Phase 1 perimeter lab with a DMZ zone and a destination NAT rule that publishes a DMZ host on the firewall's Untrust address.

## Topology
![Topology]
<img width="1910" height="955" alt="10" src="https://github.com/user-attachments/assets/15aa79b9-fd95-472e-828c-a0f45f84abe8" />


| Interface | Zone | Addressing |
|---|---|---|
| ethernet1/1 | Untrust | DHCP (VMnet8 NAT) |
| ethernet1/2 | Trust | 10.0.1.1/24 |
| ethernet1/3 | DMZ | 10.0.2.1/24 |

- PC-DMZ (VPCS): 10.0.2.10/24, gateway 10.0.2.1

![Interfaces]
<img width="1673" height="422" alt="9" src="https://github.com/user-attachments/assets/8c5c715e-9c49-4c17-bc6d-1735e5954bc4" />


![Zones]
<img width="1703" height="224" alt="8" src="https://github.com/user-attachments/assets/c7c2afa1-c71c-4740-b85c-2a9ad430872f" />



## What I configured
1. DMZ zone and interface: Layer3 interface ethernet1/3 in a new DMZ zone, with the management profile allowing ping only.
2. Security policies (ordered, with the explicit deny last):
   - Allow-Trust-to-DMZ: Trust to DMZ, only icmp, ping, traceroute, web-browsing, ssl.
   - Allow-Untrust-to-DMZ: Untrust to DMZ, destination address limited to the firewall's Untrust IP (address object FW-Untrust-IP), only icmp, ping, web-browsing, ssl.
   - Deny-All-Interzone: logged explicit deny, so the DMZ cannot initiate connections to Trust.
3. Destination NAT: Untrust-to-DMZ-DNAT translates the firewall's Untrust IP to the DMZ host 10.0.2.10.

![Security rules]
<img width="1673" height="696" alt="7" src="https://github.com/user-attachments/assets/62b1839d-ee00-4e71-b5e9-4f1a724b6237" />

![NAT rules]
<img width="1668" height="248" alt="6" src="https://github.com/user-attachments/assets/bd8d0050-9362-4330-9466-eade0e7d790e" />


## Verification
- Trust to DMZ: allowed, ping from PC-Trust to PC-DMZ succeeds.
- DMZ to Trust: blocked by Deny-All-Interzone and visible in the traffic log.
- Destination NAT: ping from PC-Attacker to the firewall's Untrust IP is translated to 10.0.2.10. The traffic log shows the NAT destination IP, and the session details show both the original and translated flows.

![Ping Trust to DMZ]
<img width="812" height="384" alt="5" src="https://github.com/user-attachments/assets/4e22858e-a64e-4249-ba99-384692af8c7b" />

![Ping DMZ to Trust blocked]
<img width="533" height="404" alt="4" src="https://github.com/user-attachments/assets/baba0908-e5a0-4ee0-8510-b705ca0da093" />

![DNAT traffic log]
<img width="1688" height="533" alt="3" src="https://github.com/user-attachments/assets/8537816d-8ad7-4b90-9de4-460a6b1255f4" />

![Deny log]
<img width="1682" height="317" alt="2" src="https://github.com/user-attachments/assets/9d67ea8a-8658-4b92-b68a-793941ed7d55" />

![Session detail]
<img width="1473" height="832" alt="1" src="https://github.com/user-attachments/assets/ff6ff284-5ac6-4b89-a589-7b153eaa8e34" />


## Notes and lessons learned
- In a NAT rule, the destination zone is the zone of the address before translation (Untrust here), while the security policy uses the zone after translation (DMZ). Using DMZ in the NAT rule at first meant it never matched and no traffic was logged.
- The Untrust address comes from DHCP, so the FW-Untrust-IP address object must be updated if the lease changes.
- The lab hosts are VPCS nodes, which only support ICMP, so the destination NAT was verified with ping, traffic logs, and session details rather than a real web service.
