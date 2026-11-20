========================================================================
SCHOOL CAMPUS NETWORK LAB - FULL TOPOLOGY DOCUMENTATION
========================================================================

1. PROJECT OVERVIEW & HIERARCHICAL ARCHITECTURE
- Topology Design: Three-tier hierarchical campus network implemented in
  Cisco Packet Tracer comprising a Core Layer (Core-Rtr), Distribution
  Layer (dual Layer 3 Catalyst 3560 multilayer switches: Dist-SW-A and
  Dist-SW-B), and Access Layer (Layer 2 access switches including
  Acces-SW-Faculty and Access-SW-Students).
- Segmentation Goal: Isolate campus departments into dedicated Layer 2
  VLAN broadcast domains and Layer 3 IPv4 subnets while providing full
  inter-VLAN and cross-building routing through redundant distribution
  gateways and dynamic OSPF routing.

2. VLAN, SUBNET & SWITCHED VIRTUAL INTERFACE (SVI) PLAN
- VLAN 10 (Faculty): 192.168.10.0/24  | SVI Gateway: 192.168.10.1 (Dist-SW-A)
- VLAN 20 (Student): 192.168.20.0/24  | SVI Gateway: 192.168.20.1 (Dist-SW-B)
- VLAN 30 (Lab):     192.168.30.0/24  | SVI Gateway: 192.168.30.1 (Dist-SW-B)
- VLAN 40 (Admin):   192.168.40.0/24  | SVI Gateway: 192.168.40.1 (Dist-SW-A)

3. LAYER 3 ROUTING & OSPF CONFIGURATION DETAILS
- Multilayer Switching: Global 'ip routing' enabled on Dist-SW-A and
  Dist-SW-B with active 'up/up' SVIs serving as default gateways.
- Point-to-Point Transit Links ('no switchport' routed interfaces):
  * Dist-SW-A (Gi0/1) <--> Core-Rtr (Gi0/1): 10.10.11.0/30
  * Dist-SW-B (Gi0/1) <--> Core-Rtr (Gi0/2): 10.10.12.0/30
- Dynamic Routing Protocol: Single-area OSPFv2 ('router ospf 1', Area 0)
  configured across Core-Rtr, Dist-SW-A, and Dist-SW-B using exact
  inverse wildcard masks ('0.0.0.3' for /30 transit links and '0.0.0.255'
  for /24 VLAN subnets). All neighbor adjacencies converged to 'FULL'
  state, dynamically exchanging 'O' routes (AD 110) across the core.

4.  ACCESS LAYER, TRUNKING & END-HOST VERIFICATION
- IEEE 802.1Q Trunking: Uplink ports between access switches and
  distribution switches configured in static trunk mode ('switchport
  mode trunk', 802.1Q encapsulation, Native VLAN 1) with VLANs 1, 10,
  20, 30, and 40 active and in Spanning Tree Protocol (STP) forwarding
  state.
- Access Port Assignments: Host-facing ports configured with 'switchport
  mode access' and bound strictly to their respective department VLANs
  (VLAN 10 Faculty, VLAN 20 Student, VLAN 30 Lab, VLAN 40 Admin).
- End-Host IPv4 Configuration: Static host IPs assigned within each
  department's /24 subnet (mask 255.255.255.0) with Default Gateways
  pointing to the owning distribution switch SVI (.1).
- Verification & Persistence: End-to-end intra-VLAN and inter-VLAN
  connectivity verified via 'ping' (0% loss) and 'tracert' across
  distribution switches and Core-Rtr. All running configurations saved
  to NVRAM via 'copy running-config startup-config'.
========================================================================
