# Phase 5: Syslog Forwarding and Site-to-Site IPsec VPN

Forwards firewall traffic logs to a syslog server in the DMZ, and builds a site-to-site IKEv2/IPsec VPN to a strongSwan peer.

## Environment
- Ubuntu in the DMZ (10.0.2.10) with rsyslog as the syslog receiver
- The same Ubuntu VM has a second NIC on the Untrust network (192.168.38.21) and acts as the remote site gateway, with a dummy interface (172.16.50.1/24) as the remote LAN, running strongSwan
- Kali client in Trust (10.0.1.10)
- Palo Alto VM-Series from the earlier phases, with no subscriptions
- The Untrust interface address was changed from DHCP to a static 192.168.38.20/24 for the VPN, so screenshots from Phase 2 show the earlier DHCP address.

## Part 1: Syslog forwarding
### What I configured
- rsyslog on Ubuntu listens on UDP 514 and writes messages from the firewall (10.0.2.1) to /var/log/paloalto.log.
- Syslog server profile Ubuntu-Syslog on the firewall (UDP 514).
- A service route for Syslog through ethernet1/3, because the firewall sends syslog from the management interface by default and the server is in the DMZ.
- Log forwarding profile Forward-to-Syslog, attached to the security rules.

![Syslog profile]

<img width="1675" height="174" alt="3" src="https://github.com/user-attachments/assets/8e9a10cb-825f-44e5-9859-30f2b68e286d" />

![Service route]

<img width="621" height="378" alt="9" src="https://github.com/user-attachments/assets/2e98cd32-5fe3-4a9e-b9f5-94905162fab5" />

![Log forwarding]

<img width="1676" height="302" alt="21" src="https://github.com/user-attachments/assets/6e1ef8b2-2ab6-42ae-a8ee-39c50e030262" />


### Verification
- After generating traffic from Kali, traffic log entries appear in /var/log/paloalto.log on Ubuntu, including the rule names and applications.

![Syslog received]

<img width="1718" height="416" alt="11" src="https://github.com/user-attachments/assets/fb045054-6b81-452b-974e-fe40b5b6ad07" />


## Part 2: Site-to-site IPsec VPN
### What I configured
- Tunnel interface tunnel.1 in a new VPN zone.
- IKE crypto profile: AES-256-CBC, SHA-256, DH group 14. IPsec crypto profile: ESP with AES-256-CBC and SHA-256, no PFS.
- IKE gateway GW-Ubuntu (IKEv2, pre-shared key, not published here) and IPsec tunnel Tunnel-Ubuntu, with a proxy ID of 10.0.1.0/24 to 172.16.50.0/24.
- Static route to 172.16.50.0/24 through tunnel.1.
- Security rules Allow-Trust-to-VPN and Allow-VPN-to-Trust (icmp and ping only).
- strongSwan on Ubuntu with matching IKEv2 and ESP settings.

![NAT rules]

<img width="1682" height="281" alt="12" src="https://github.com/user-attachments/assets/e7a61f11-44b6-49f7-a132-44c1e59b7d50" />


![IKE gateway]

<img width="1681" height="149" alt="5" src="https://github.com/user-attachments/assets/2a96f3b0-45d2-4f66-a7a6-18dd01df1458" />

![IPsec tunnel]

<img width="1683" height="179" alt="14" src="https://github.com/user-attachments/assets/d21f0b38-9899-4f15-bdb3-a5c42bddfde3" />



![Proxy IDs]

<img width="1077" height="323" alt="15" src="https://github.com/user-attachments/assets/d127669e-a96b-4521-8eee-c29072a504b1" />

![VPN rules]

<img width="1696" height="166" alt="16" src="https://github.com/user-attachments/assets/6e7fed99-246b-4792-987c-da8f72c19203" />

![strongSwan config]

<img width="521" height="277" alt="17" src="https://github.com/user-attachments/assets/b952d09a-6b7a-47a0-8ca1-30f3b36086f6" />


### Verification
- Ping from Kali (Trust) to 172.16.50.1 goes through the tunnel and succeeds.
- strongSwan shows the IKE SA as ESTABLISHED and the child SA as INSTALLED, with the byte counters increasing after the ping.
- The firewall traffic log shows the ping matching Allow-Trust-to-VPN toward the VPN zone, and the tunnel status is green.

![Ping through tunnel]

<img width="1125" height="82" alt="20" src="https://github.com/user-attachments/assets/fb9dddb4-fa83-4959-a819-baafce01960a" />

![strongSwan status]

<img width="1723" height="432" alt="19" src="https://github.com/user-attachments/assets/c20c1db8-6604-4146-ba51-feded6e2425e" />

![Traffic log]

<img width="1676" height="302" alt="21" src="https://github.com/user-attachments/assets/3a3c3c7c-3704-4eff-aa46-126069cf6b79" />


## Troubleshooting notes
- The tunnel stayed in "connecting" at first. The Destination NAT rule from Phase 2 translated any traffic arriving at the firewall's Untrust address, including IKE (UDP 500), to the DMZ server, so the firewall never saw the IKE packets. I added a no-NAT rule for the VPN peer above the Destination NAT rule, and the tunnel came up. A successful ping to the firewall address was misleading, because the reply came from the DMZ server.
- Syslog and the management interface. By default the firewall sends syslog from the management interface, so a service route was needed for a server in the DMZ.
- A typo in the rsyslog configuration (the module line and the input line ran together) left rsyslog running without a listener. Rewriting the file fixed it.
- [Add any other problem you fixed.]

## Limitations
- The "remote site" is a Linux VM on the same host with a dummy interface as its LAN, not a separate network.
- Authentication uses a pre-shared key. Certificates were not used.
- The lab has no Palo Alto subscriptions.
