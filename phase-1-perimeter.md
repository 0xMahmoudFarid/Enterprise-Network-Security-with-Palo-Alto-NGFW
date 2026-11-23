# Palo Alto VM-Series Firewall Lab (GNS3 + VMware)

Hands-on lab of a perimeter firewall using Palo Alto VM-Series, built with only GNS3, the GNS3 VM, and VMware Workstation.

Status: Phase 1 complete. Next phases are planned below.

## Tools
- Palo Alto VM-Series (PAN-OS 11.1.6-h4)
- VMware Workstation (virtual networks: VMnet2 = Trust, VMnet8 = Untrust)
- GNS3 + GNS3 VM (Cloud node, Ethernet switch, VPCS hosts)

## Topology
![Topology](screenshots/01-topology.png)

____________________________________________________
|  Interface  |    Zone   |       Addressing       |
|-------------|-----------|------------------------|
| ethernet1/1 |  Untrust  |   DHCP (VMnet8 NAT)    |
| ethernet1/2 |   Trust   |      10.0.1.1/24       |
|__________________________________________________|


- PC-Trust (VPCS): 10.0.1.10/24, gateway 10.0.1.1
- PC-Attacker (VPCS): placed in the Untrust network

## What I configured
1. Zones and interfaces: Layer3 interfaces assigned to Trust and Untrust zones, with a management profile allowing ping only.
2. Routing: Virtual router default with a default route to the Untrust gateway.
3. Source NAT: Trust to Untrust, dynamic IP and port using the Untrust interface address.
4. Security policies:
   - Allow-Trust-to-Untrust: allows only specific applications (icmp, ping, traceroute, dns, web-browsing, ssl), logging at session end.
   - Deny-All-Interzone: explicit deny with logging for all other inter-zone traffic.

![Interfaces](screenshots/02-interfaces.png)
![Zones](screenshots/03-zones.png)
![Route table](screenshots/04-route-table.png)
![NAT rule](screenshots/05-nat-rule.png)
![Security rules](screenshots/06-security-rules.png)

## Verification
- Allowed traffic: PC-Trust reaches 8.8.8.8. Traffic logs show allow with the translated source IP.
- Blocked traffic: ping from PC-Attacker (Untrust) to PC-Trust is dropped and logged under Deny-All-Interzone.
- Session table: shows the NAT translation for the active session.

![Ping test](screenshots/07-ping-test.png)
![Allow log](screenshots/08-traffic-allow.png)
![Deny log](screenshots/09-traffic-deny.png)
![Session](screenshots/10-session.png)

## Roadmap
- [ ] DMZ zone and destination NAT (publish an internal server)
- [ ] App-ID and User-ID policies
- [ ] SSL decryption and threat prevention
- [ ] Site-to-site VPN, HA, and log forwarding to a SIEM