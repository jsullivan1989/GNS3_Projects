# EVPN VXLAN Project - Centrally Routed Bridging

This GNS3 project outlines a simple two-leaf, two-spine topology in which the spines serve as the L3 VXLAN Gateway.  The Leafs in this example are layer 2 only while the Spines run VGW for VLANs 10 and 20.  
The VGW IP addresses are 192.168.10.1/24 (VLAN 10) and 192.168.20.1/24 (VLAN 20) respectively.  eBGP is used as both the underlay and overlay protocol in order to capture the common differences between the setups.
The underlay eBGP peerings were done via direct connections allocated in the 10.0.0.0/29 range.  The overlay eBGP peerings were done via Loopback peering.  Each leaf and spine has its own unique overlay/underlay ASN.







## Topology

![image](https://github.com/user-attachments/assets/4831cee5-2a2d-4a7a-87b7-1fd1bf9faa62)


