# Phase 2: DMZ and Destination NAT

Extends the Phase 1 perimeter lab with a DMZ zone and a destination NAT rule that publishes a DMZ host on the firewall's Untrust address.

## Topology
![Topology](screenshots/01-topology.png)

| Interface | Zone | Addressing |
|---|---|---|
| ethernet1/1 | Untrust | DHCP (VMnet8 NAT) |
| ethernet1/2 | Trust | 10.0.1.1/24 |
| ethernet1/3 | DMZ | 10.0.2.1/24 |

- PC-DMZ (VPCS): 10.0.2.10/24, gateway 10.0.2.1

![Interfaces](screenshots/02-interfaces.png)
![Zones](screenshots/03-zones.png)

## What I configured
1. DMZ zone and interface: Layer3 interface ethernet1/3 in a new DMZ zone, with the management profile allowing ping only.
2. Security policies (ordered, with the explicit deny last):
   - Allow-Trust-to-DMZ: Trust to DMZ, only icmp, ping, traceroute, web-browsing, ssl.
   - Allow-Untrust-to-DMZ: Untrust to DMZ, destination address limited to the firewall's Untrust IP (address object FW-Untrust-IP), only icmp, ping, web-browsing, ssl.
   - Deny-All-Interzone: logged explicit deny, so the DMZ cannot initiate connections to Trust.
3. Destination NAT: Untrust-to-DMZ-DNAT translates the firewall's Untrust IP to the DMZ host 10.0.2.10.

![Security rules](screenshots/04-security-rules.png)
![NAT rules](screenshots/05-nat-rules.png)

## Verification
- Trust to DMZ: allowed, ping from PC-Trust to PC-DMZ succeeds.
- DMZ to Trust: blocked by Deny-All-Interzone and visible in the traffic log.
- Destination NAT: ping from PC-Attacker to the firewall's Untrust IP is translated to 10.0.2.10. The traffic log shows the NAT destination IP, and the session details show both the original and translated flows.

![Ping Trust to DMZ](screenshots/06-ping-trust-dmz.png)
![Ping DMZ to Trust blocked](screenshots/07-ping-dmz-trust-blocked.png)
![DNAT traffic log](screenshots/08-traffic-dnat.png)
![Deny log](screenshots/09-traffic-deny.png)
![Session detail](screenshots/10-session-dnat.png)

## Notes and lessons learned
- In a NAT rule, the destination zone is the zone of the address before translation (Untrust here), while the security policy uses the zone after translation (DMZ). Using DMZ in the NAT rule at first meant it never matched and no traffic was logged.
- The Untrust address comes from DHCP, so the FW-Untrust-IP address object must be updated if the lease changes.
- The lab hosts are VPCS nodes, which only support ICMP, so the destination NAT was verified with ping, traffic logs, and session details rather than a real web service.