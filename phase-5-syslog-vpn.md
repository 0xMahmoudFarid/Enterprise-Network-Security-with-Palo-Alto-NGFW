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

![Syslog profile](screenshots/01-syslog-profile.png)
![Service route](screenshots/02-service-route.png)
![Log forwarding](screenshots/03-log-forwarding.png)

### Verification
- After generating traffic from Kali, traffic log entries appear in /var/log/paloalto.log on Ubuntu, including the rule names and applications.

![Syslog received](screenshots/04-syslog-received.png)

## Part 2: Site-to-site IPsec VPN
### What I configured
- Tunnel interface tunnel.1 in a new VPN zone.
- IKE crypto profile: AES-256-CBC, SHA-256, DH group 14. IPsec crypto profile: ESP with AES-256-CBC and SHA-256, no PFS.
- IKE gateway GW-Ubuntu (IKEv2, pre-shared key, not published here) and IPsec tunnel Tunnel-Ubuntu, with a proxy ID of 10.0.1.0/24 to 172.16.50.0/24.
- Static route to 172.16.50.0/24 through tunnel.1.
- Security rules Allow-Trust-to-VPN and Allow-VPN-to-Trust (icmp and ping only).
- strongSwan on Ubuntu with matching IKEv2 and ESP settings.

![NAT rules](screenshots/05-nat-rules.png)
![IKE gateway](screenshots/06-ike-gateway.png)
![IPsec tunnel](screenshots/07-ipsec-tunnel-green.png)
![Proxy IDs](screenshots/08-proxy-ids.png)
![VPN rules](screenshots/09-vpn-security-rules.png)
![strongSwan config](screenshots/10-strongswan-conf.png)

### Verification
- Ping from Kali (Trust) to 172.16.50.1 goes through the tunnel and succeeds.
- strongSwan shows the IKE SA as ESTABLISHED and the child SA as INSTALLED, with the byte counters increasing after the ping.
- The firewall traffic log shows the ping matching Allow-Trust-to-VPN toward the VPN zone, and the tunnel status is green.

![Ping through tunnel](screenshots/11-ping-through-tunnel.png)
![strongSwan status](screenshots/12-ipsec-statusall.png)
![Traffic log](screenshots/13-traffic-log-vpn.png)

## Troubleshooting notes
- The tunnel stayed in "connecting" at first. The Destination NAT rule from Phase 2 translated any traffic arriving at the firewall's Untrust address, including IKE (UDP 500), to the DMZ server, so the firewall never saw the IKE packets. I added a no-NAT rule for the VPN peer above the Destination NAT rule, and the tunnel came up. A successful ping to the firewall address was misleading, because the reply came from the DMZ server.
- Syslog and the management interface. By default the firewall sends syslog from the management interface, so a service route was needed for a server in the DMZ.
- A typo in the rsyslog configuration (the module line and the input line ran together) left rsyslog running without a listener. Rewriting the file fixed it.
- [Add any other problem you fixed.]

## Limitations
- The "remote site" is a Linux VM on the same host with a dummy interface as its LAN, not a separate network.
- Authentication uses a pre-shared key. Certificates were not used.
- The lab has no Palo Alto subscriptions.