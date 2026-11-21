========================================================================
CISCO PACKET TRACER LAB DOCUMENTATION - COMPLETE START-TO-FINISH RECORD
========================================================================
1. PROJECT EXPLANATION & OBJECTIVE
- Build, configure, and verify a home network connected to an ISP edge
  in Cisco Packet Tracer, covering DHCP address allocation, local DNS
  resolution (A records), default gateway routing (192.168.1.1), static
  default routing (0.0.0.0/0), and Port Address Translation (PAT / NAT
  overload) across a point-to-point WAN link.
- Enables internal LAN endpoints (PC, Laptop) and a local DNS server on
  192.168.1.0/24 to communicate locally and reach the ISP router
  (10.10.10.2) and simulated external target (8.8.8.8) using a single
  outside IPv4 address (10.10.10.1) via dynamic port translation.

2. TOPOLOGY, CABLING & IP ADDRESSING PLAN
- Home LAN Subnet: 192.168.1.0/24 | Mask: 255.255.255.0
  * Home Router LAN Interface (Fa0/0): 192.168.1.1 (Default Gateway)
  * DNS Server: 192.168.1.2 (Static IP, GW 192.168.1.1, DNS & A Record)
  * PC / Laptop: DHCP client (192.168.1.x, GW 192.168.1.1, DNS 192.168.1.2)
- Point-to-Point WAN Subnet: 10.10.10.0/30 | Mask: 255.255.255.252
  * Home Router WAN Interface (Fa1/0): 10.10.10.1 (NAT Outside)
  * ISP Router WAN Interface (Fa0/0 or Fa0/1 per active port): 10.10.10.2
  * Simulated External Target on ISP Router (Loopback0): 8.8.8.8/32
- Cabling: Copper Straight-Through between LAN hosts/server and switch,
  and between switch and Home Router LAN port; Copper Crossover between
  Home Router WAN port and ISP Router WAN port.

3. ROUTING, DHCP, DNS & NAT/PAT ARCHITECTURE
- Default Gateway & Default Route: LAN clients send off-subnet traffic
  to 192.168.1.1; Home Router forwards 0.0.0.0/0 via next hop
  10.10.10.2 using 'ip route 0.0.0.0 0.0.0.0 10.10.10.2'.
- NAT Overload (PAT): Standard ACL matches 192.168.1.0/24; 'ip nat
  inside' on Fa0/0 (LAN) and 'ip nat outside' on Fa1/0 (WAN) activate
  'ip nat inside source list <ACL> interface Fa1/0 overload' so multiple
  private LAN hosts share 10.10.10.1 via unique source ports.
- DHCP & DNS Services: DHCP pool assigns IPv4 address, subnet mask,
  default router (192.168.1.1), and DNS server (192.168.1.2) while
  excluding static infrastructure IPs; DNS server resolves domain A
  records to IPv4 addresses.

4. COMPLETE CISCO IOS CLI CONFIGURATION COMMANDS
- Home Router (LAN Gateway, DHCP, NAT/PAT & Default Route):
  enable
  configure terminal
  hostname Home-Router
  ip dhcp excluded-address 192.168.1.1 192.168.1.10
  ip dhcp pool HOME_LAN_POOL
   network 192.168.1.0 255.255.255.0
   default-router 192.168.1.1
   dns-server 192.168.1.2
  exit
  interface FastEthernet0/0
   description Home LAN Default Gateway
   ip address 192.168.1.1 255.255.255.0
   ip nat inside
   duplex auto
   speed auto
   no shutdown
  exit
  interface FastEthernet1/0
   description WAN Link to ISP Router
   ip address 10.10.10.1 255.255.255.252
   ip nat outside
   duplex auto
   speed auto
   no shutdown
  exit
  ip access-list standard LAN_NAT_ACL
   permit 192.168.1.0 0.0.0.255
  exit
  ip nat inside source list LAN_NAT_ACL interface FastEthernet1/0 overload
  ip route 0.0.0.0 0.0.0.0 10.10.10.2
  end
  copy running-config startup-config

- ISP Router (WAN Link & Simulated External Loopback):
  enable
  configure terminal
  hostname ISP-Router
  interface FastEthernet0/0
   description WAN Link from Home Router
   ip address 10.10.10.2 255.255.255.252
   duplex auto
   speed auto
   no shutdown
  exit
  interface Loopback0
   description Simulated External Internet Target
   ip address 8.8.8.8 255.255.255.255
   no shutdown
  exit
  end
  copy running-config startup-config

5. VERIFICATION & TROUBLESHOOTING COMMANDS
- Router Interface & Routing Checks:
  show ip interface brief
  show running-config
  show running-config | begin interface FastEthernet0/0
  show ip route
- DHCP & NAT/PAT Checks:
  show ip dhcp pool
  show ip dhcp binding
  show ip nat translations
  show ip nat statistics
- Switch & Endpoint Checks:
  show mac address-table
  ipconfig /all
  ping 192.168.1.1
  ping 192.168.1.2
  ping 10.10.10.2
  ping 8.8.8.8
  nslookup <domain-name>
========================================================================
