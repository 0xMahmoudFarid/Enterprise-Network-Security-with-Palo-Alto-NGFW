# Palo Alto VM-Series Firewall Lab (GNS3 + VMware)

Hands-on lab of a perimeter firewall using Palo Alto VM-Series, built with only GNS3, the GNS3 VM, and VMware Workstation.

Status: Phase 1 complete. Next phases are planned below.

## Tools
- Palo Alto VM-Series (PAN-OS 11.1.6-h4)
- VMware Workstation (virtual networks: VMnet2 = Trust, VMnet8 = Untrust)
- GNS3 + GNS3 VM (Cloud node, Ethernet switch, VPCS hosts)

## Topology
![Topology]
<img width="1916" height="959" alt="01-topology" src="https://github.com/user-attachments/assets/d68640c7-e5ee-4bdf-890b-78e20748b7df" />


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

![Interfaces]
<img width="1533" height="419" alt="02-interfaces" src="https://github.com/user-attachments/assets/c5cc3be1-bc21-4f1d-9418-7370b4a94e8f" />

![Zones]
<img width="1580" height="178" alt="03-zones" src="https://github.com/user-attachments/assets/cdb51057-c1d7-46b4-bc86-01bf5fef4b3d" />

![Route table]
<img width="996" height="400" alt="04-route-table" src="https://github.com/user-attachments/assets/1d92b3cf-7672-49cf-92b1-549368fd7e6f" />

![NAT rule]
<img width="1427" height="215" alt="05-nat-rule" src="https://github.com/user-attachments/assets/0a275e04-f0b7-4d8d-a34d-f763def550ad" />

![Security rules]
<img width="1667" height="433" alt="06-security-rules" src="https://github.com/user-attachments/assets/395c6162-d29b-4c56-975e-d9f6639136d9" />

## Verification
- Allowed traffic: PC-Trust reaches 8.8.8.8. Traffic logs show allow with the translated source IP.
- Blocked traffic: ping from PC-Attacker (Untrust) to PC-Trust is dropped and logged under Deny-All-Interzone.
- Session table: shows the NAT translation for the active session.

![Ping test]
<img width="743" height="608" alt="07-ping-test" src="https://github.com/user-attachments/assets/6d9582cc-a284-46ab-9f1f-1cdc956cbb94" />

![Allow log]
<img width="1680" height="188" alt="08-traffic-deny" src="https://github.com/user-attachments/assets/7be0f9e0-4b47-451a-8ec6-95b7947de8aa" />

![Deny log]
<img width="1680" height="188" alt="09-traffic-deny" src="https://github.com/user-attachments/assets/ae145612-1184-4b45-86db-00d738408d07" />

![Session]
<img width="1675" height="413" alt="10-session" src="https://github.com/user-attachments/assets/fbba222a-34db-4a04-b73f-6f5c7545e183" />


## Roadmap
- [ ] DMZ zone and destination NAT (publish an internal server)
- [ ] App-ID and User-ID policies
- [ ] SSL decryption and threat prevention
- [ ] Site-to-site VPN, HA, and log forwarding to a SIEM
