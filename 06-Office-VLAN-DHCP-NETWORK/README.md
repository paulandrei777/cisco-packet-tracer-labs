========================================================================
SINGLE-BUILDING MULTI-VLAN & CENTRALIZED DHCP LAB - FULL DOCUMENTATION
========================================================================
1. PROJECT EXPLANATION & SCOPE
- Scenario: Single-building office/enterprise network in Cisco Packet
  Tracer segmented into three departments:
    * IT Department (VLAN 10)      - at least 5 end devices
    * HR Department (VLAN 20)      - at least 5 end devices
    * Accounting Dept (VLAN 30)  - at least 5 end devices
- Architecture: Two Layer 2 switches handle access port VLAN assignment
  and 802.1Q trunking, while a central router provides per-VLAN
  subinterface default gateways, inter-VLAN routing, and centralized
  Cisco IOS DHCP pools. All client hosts obtain IPv4 addresses dynamically
  via DHCP within each VLAN's configured subnet and exclusion range.

2. IPv4 SUBNETTING, VLAN & DHCP DESIGN PLAN
- VLAN 10 (IT):         192.168.10.0/24  (Mask: 255.255.255.0)
                        Gateway: .10.1 | Usable Hosts: .10.1-.10.254
- VLAN 20 (HR):         192.168.20.0/24  (Mask: 255.255.255.0)
                        Gateway: .20.1 | Usable Hosts: .20.1-.20.254
- VLAN 30 (Accounting): 192.168.30.0/24  (Mask: 255.255.255.0)
                        Gateway: .30.1 | Usable Hosts: .30.1-.30.254
  
- DHCP Exclusions & Pools: Each 'ip dhcp pool' on the router defines
  'network <subnet> <mask>' and 'default-router <gateway>', while
  'ip dhcp excluded-address' reserves gateway IPs and any static or
  excluded address range from dynamic leasing.

3. ROUTING, SWITCHING & CLI BEHAVIOR
- Layer 2 Switching: Access ports use 'switchport mode access' and
  'switchport access vlan <id>' to isolate broadcast domains per
  department. Inter-switch and switch-to-router uplinks use
  'switchport mode trunk' (802.1Q) to carry tagged frames for VLANs
  10, 20, and 30.
- Router-on-a-Stick & Routing Table: The physical router interface
  connected to the switch trunk is brought up with 'no shutdown', and
  subinterfaces (e.g., Gi0/0.10, Gi0/0.20, Gi0/0.30) are configured with
  'encapsulation dot1q <vlan-id>' and their respective '.1' IP
  addresses. IOS automatically installs directly connected ('C') and
  local ('L') routes for each subnet in 'show ip route', enabling
  inter-VLAN packet forwarding across IT, HR, and Accounting.
- Key Verification Commands:
  * Switches: 'show vlan brief', 'show interfaces trunk',
              'show mac address-table'
  * Router:   'show ip interface brief', 'show ip route',
              'show ip dhcp binding', 'show ip dhcp pool'
  * End PCs:  IPv4 set to DHCP; verify IP/mask/gateway and test 'ping'
              within the same VLAN and across different VLANs.

4. EXECUTION PHASES
- Phase 1: Place router, 2 switches, and 5+ PCs per dept (15+ total);
           cable devices and name them clearly.
- Phase 2: Create VLANs 10, 20, 30; assign access ports; configure
           802.1Q trunk ports.
- Phase 3: Enable physical router interface and configure 802.1Q
           subinterfaces (.10, .20, .30) with '.1' gateway IPs.
- Phase 4: Configure 'ip dhcp excluded-address' and 'ip dhcp pool'
           entries for VLANs 10, 20, and 30 on the router.
- Phase 5: Switch PCs to DHCP, verify leases and 'show ip route', test
           end-to-end 'ping', and save configs ('copy running-config 
           startup-config').
========================================================================
