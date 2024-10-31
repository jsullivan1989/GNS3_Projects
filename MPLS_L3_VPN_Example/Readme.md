# MPLS Layer 3 VPN Project

This GNS3 Project was created to showcase a simple MPLS Layer 3 VPN with Junos running LDP and using OSPF as the underlay network.  Both PE routers are running internal BGP over AS100 and peering with
their loopback IP addresses.  In addition, each PE router has one directly connected interface in the Customer-A and Customer-B VRFs respectively.  The vJunos-router-23.2R1.15.qcow2 image was used to 
stand up the Junos Router VMs.

## Topology

<img width="783" alt="Screenshot 2024-10-28 at 9 59 02 PM" src="https://github.com/user-attachments/assets/06ac275e-0eca-45d1-ae57-92c7a4441cae">


## Qemu VM Settings

<img width="1728" alt="Screenshot 2024-10-28 at 10 04 59 PM" src="https://github.com/user-attachments/assets/822defbb-1a89-43f0-96b6-b1605ec8dbe8">


## PE1 Router Configuration

The PE1 router (hostname: jsullivan@EVO-Sullivan-PE1) has three physical and one loopback interface in use.

1. Interface ge-0/0/4 is attached to interface ge-0/0/4 on the P Router.  This interface is routed with the IP address 10.0.0.2/30 and assigned to OSPF Area 0.  In addition, the below configuration shows
   how to enable the interface to run both MPLS and LDP.

```
jsullivan@EVO-Sullivan-PE1> show interfaces terse | match ge-0/0/4 
ge-0/0/4                up    up
ge-0/0/4.0              up    up   inet     10.0.0.2/30

jsullivan@EVO-Sullivan-PE1> show configuration interfaces ge-0/0/4  
unit 0 {
    family inet {
        address 10.0.0.2/30;
    }
    family mpls;
}

jsullivan@EVO-Sullivan-PE1> show configuration protocols mpls 
interface ge-0/0/4.0;

jsullivan@EVO-Sullivan-PE1> show configuration protocols ldp     
interface ge-0/0/4.0;
interface lo0.0;

jsullivan@EVO-Sullivan-PE1> show configuration protocols ospf area 0   
interface ge-0/0/4.0;
interface lo0.0;

```

2. Interface ge-0/0/5 is a part of the Customer-A VRF and is directly attached to PC2 in the diagram.  Interface ge-0/0/5 (192.168.10.1/24) is the gateway for the PC and is assigned to the Customer-A VRF

```

jsullivan@EVO-Sullivan-PE1> show configuration interfaces ge-0/0/5 
unit 0 {
    family inet {
        address 192.168.10.1/24;
    }
}

jsullivan@EVO-Sullivan-PE1> show configuration routing-instances Customer-A | match interface    
interface ge-0/0/5.0;

```

3. Interface ge-0/0/6 is a part of the Customer-B VRF and is directly attached to PC1 in the diagram.  Interface ge-0/0/6 (192.168.20.1/24) is the gateway for the PC and is assigned to the Customer-2 VRF

```

jsullivan@EVO-Sullivan-PE1> show configuration interfaces ge-0/0/6 
unit 0 {
    family inet {
        address 192.168.20.1/24;
    }
}

jsullivan@EVO-Sullivan-PE1> show configuratino routing-instances Customer-B | match ge-0/0/6      
interface ge-0/0/6.0;


```

4. Finally, the Loopback Interface is configured with the host IP 192.168.1.1/32.  The Loopback is also assigned to Area 0, so that PE2 will have reachability for its internal BGP peering

```
jsullivan@EVO-Sullivan-PE1> show configuration interfaces lo0 
unit 0 {
    family inet {
        address 192.168.1.1/32;
    }
}

jsullivan@EVO-Sullivan-PE1> show configuration protocols ospf area 0 | match lo0
interface lo0.0;

jsullivan@EVO-Sullivan-PE1> show configuration protocols ospf area 0 | match lo0                
interface lo0.0;

jsullivan@EVO-Sullivan-PE1> show configuration protocols ldp | match lo0
interface lo0.0;


```

5. The BGP session is set to peer with PE2's local loopback address and set with family inet-vpn so that BGP can carry IPv4 NLRI for our VPN routes.

```

jsullivan@EVO-Sullivan-PE1> show configuration protocols bgp 
group MPLS-IBGP {
    local-address 192.168.1.1;
    neighbor 192.168.1.2 {
        family inet-vpn {
            unicast;
        }
        peer-as 100;
    }
}

```

6. Both routing-instances Customer-A and Customer-B are created with the VRF instance-type since this is a Layer 3 VPN Deployment.  The respective interfaces are also assigned to the appropriate routing-instances
   under this stanza.  We also need to define and assign a Route-Distinguisher to the VRF as well as Route Targets.  The "vrf-target" command in Junos is a shortcut to both import and export the vrf target communities
   that are defined afterward.  Lastly, the command "vrf-table-label" is included in order to map an inner VPN label to our VRF.  The label is created automatically along with an LSI (Label-Switched Interface).

```

jsullivan@EVO-Sullivan-PE1> show configuration routing-instances Customer-A                     
instance-type vrf;
interface ge-0/0/5.0;
route-distinguisher 192.168.1.1:100;
vrf-target target:100:100;
vrf-table-label;

jsullivan@EVO-Sullivan-PE1> show configuration routing-instances Customer-B    
instance-type vrf;
interface ge-0/0/6.0;
route-distinguisher 192.168.1.1:200;
vrf-target target:200:200;
vrf-table-label;

```

## PE2 Router Configuration

1. Interface ge-0/0/4 is attached to interface ge-0/0/5 on the P Router for the OSPF connection.  This interface is routed with the IP address 10.0.0.6/30 and assigned to OSPF Area 0.  The below configuration shows
   how to enable the interface to run both MPLS and LDP.

```
jsullivan@EVO-Sullivan-PE2> show interfaces terse | match ge-0/0/4 
ge-0/0/4                up    up
ge-0/0/4.0              up    up   inet     10.0.0.6/30

jsullivan@EVO-Sullivan-PE2> show configuration interfaces ge-0/0/4  
unit 0 {
    family inet {
        address 10.0.0.6/30;
    }
    family mpls;
}

jsullivan@EVO-Sullivan-PE2> show configuration protocols mpls 
interface ge-0/0/4.0;

jsullivan@EVO-Sullivan-PE2> show configuration protocols ldp     
interface ge-0/0/4.0;
interface lo0.0;

jsullivan@EVO-Sullivan-PE2> show configuration protocols ospf area 0   
interface ge-0/0/4.0;
interface lo0.0;

```

2. Interface ge-0/0/5 is a part of the Customer-A VRF and is directly attached to PC3 in the diagram.  Interface ge-0/0/5 (192.168.30.1/24) is the gateway for the PC and is assigned to the Customer-A VRF.  

```

jsullivan@EVO-Sullivan-PE2> show configuration interfaces ge-0/0/5 
unit 0 {
    family inet {
        address 192.168.30.1/24;
    }
}

jsullivan@EVO-Sullivan-PE2> show configuration routing-instances Customer-A | match interface    
interface ge-0/0/5.0;

```

3. Interface ge-0/0/6 is a part of the Customer-B VRF and is directly attached to PC4 in the diagram.  Interface ge-0/0/6 (192.168.40.1/24) is the gateway for the PC and is assigned to the Customer-2 VRF

```

jsullivan@EVO-Sullivan-PE2> show configuration interfaces ge-0/0/6 
unit 0 {
    family inet {
        address 192.168.40.1/24;
    }
}

jsullivan@EVO-Sullivan-PE2> show configuratino routing-instances Customer-B | match ge-0/0/6      
interface ge-0/0/6.0;


```

4. The Loopback Interface is configured with the host IP 192.168.1.2/32.  The Loopback is also assigned to Area 0, so that PE2 will have reachability for its internal BGP peering with Router 1's Loopback 192.168.1.1/32 mentioned earlier.

```
jsullivan@EVO-Sullivan-PE2> show configuration interfaces lo0 
unit 0 {
    family inet {
        address 192.168.1.2/32;
    }
}

jsullivan@EVO-Sullivan-PE2> show configuration protocols ospf area 0 | match lo0
interface lo0.0;

jsullivan@EVO-Sullivan-PE2> show configuration protocols ospf area 0 | match lo0                
interface lo0.0;

jsullivan@EVO-Sullivan-PE2> show configuration protocols ldp | match lo0
interface lo0.0;


```

5. The BGP session is set to peer with PE1's local loopback address of 192.168.1.1 and set with family inet-vpn so that BGP can carry IPv4 NLRI for our VPN routes.

```

jsullivan@EVO-Sullivan-PE2> show configuration protocols bgp 
group MPLS-IBGP {
    local-address 192.168.1.2;
    neighbor 192.168.1.1 {
        family inet-vpn {
            unicast;
        }
        peer-as 100;
    }
}

```

6. Both routing-instances Customer-A and Customer-B are also configured below in a similar fashion as PE1.  A Route-Distinguisher and Route Targets are assigned along with the "vrf-table-label" knob just like PE1.

```

jsullivan@EVO-Sullivan-PE2> show configuration routing-instances Customer-A                      
instance-type vrf;
interface ge-0/0/5.0;
route-distinguisher 192.168.1.2:100;
vrf-target target:100:100;
vrf-table-label;

jsullivan@EVO-Sullivan-PE2> show configuration routing-instances Customer-B    
instance-type vrf;
interface ge-0/0/6.0;
route-distinguisher 192.168.1.2:200;
vrf-target target:200:200;
vrf-table-label;

```

## P Router Configuration

The P-Router has a simple configuration for this deployment.  We assign interfaces ge-0/0/4 and ge-0/0/5 to OSPF Area 0, so that it can form OSPF neighborships with PE1 and PE1 over the respective 10.0.0.0/30 and 10.0.0.4/30 networks as mentioned earlier.  The Loopback address 192.168.1.3/32 is also assigned to OSPF area 0 so that PE1 and PE2 can build LDP sessions with the P-Router.  The P-Router is not running BGP.

```
jsullivan@EVO-Sullivan-P-Router> show configuration interfaces ge-0/0/4 
unit 0 {
    family inet {
        address 10.0.0.1/30;
    }
    family mpls;
}

jsullivan@EVO-Sullivan-P-Router> show configuration interfaces ge-0/0/5    
unit 0 {
    family inet {
        address 10.0.0.5/30;
    }
    family mpls;
}

jsullivan@EVO-Sullivan-P-Router> show configuration interfaces lo0 
unit 0 {
    family inet {
        address 192.168.1.3/32;
    }
}

```

The MPLS and LDP configuration on the P-Router is shown below.

```

jsullivan@EVO-Sullivan-P-Router> show configuration protocols mpls 
interface ge-0/0/4.0;
interface ge-0/0/5.0;


jsullivan@EVO-Sullivan-P-Router> show configuration protocols ldp     
interface ge-0/0/4.0;
interface ge-0/0/5.0;
interface lo0.0;

```

