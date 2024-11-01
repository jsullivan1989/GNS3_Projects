# Network Validation

<img width="783" alt="Screenshot 2024-10-28 at 9 59 02 PM" src="https://github.com/user-attachments/assets/fc60a04f-2543-49da-8900-d110907d2341">


Once the network is in place, there are several validations that need to take place to verify end to end connectivity between the PCs in their respective Customer VRFs.
The expected behavior is for the Customer A VRF networks 192.168.10.0/24 and 192.168.30.0/24 to communicate as well as Customer B's 192.168.20.0/24 and 192.168.40.0/24 networks.

Is that the case?

First we try to ping from PC1 (192.168.10.10) to PC3 (192.168.30.10).  In GNS3, you assign a host address on a PC with the command 
ip <address> <dotted decimal mask> <ip gateway>

## PC2 Network Configuration

```

PC2> ip 192.168.10.10 255.255.255.0 192.168.10.1
Checking for duplicate address...
PC2 : 192.168.10.10 255.255.255.0 gateway 192.168.10.1

# pinging the gateway, PE1

PC2> ping 192.168.10.1

84 bytes from 192.168.10.1 icmp_seq=1 ttl=64 time=1.126 ms
84 bytes from 192.168.10.1 icmp_seq=2 ttl=64 time=0.555 ms
84 bytes from 192.168.10.1 icmp_seq=3 ttl=64 time=0.325 ms
^C
```

## PC3 Network Configuration

```

PC3> ip 192.168.30.10 255.255.255.0 192.168.30.1
Checking for duplicate address...
PC3 : 192.168.30.10 255.255.255.0 gateway 192.168.30.1


PC3> ping 192.168.30.1 

84 bytes from 192.168.30.1 icmp_seq=1 ttl=64 time=0.361 ms
84 bytes from 192.168.30.1 icmp_seq=2 ttl=64 time=0.410 ms
84 bytes from 192.168.30.1 icmp_seq=3 ttl=64 time=0.406 ms
84 bytes from 192.168.30.1 icmp_seq=4 ttl=64 time=0.347 ms
```

Now let's do the same for the PCs that belong to Customer B (PC1 and PC4

## PC1 Network Configuration
```

PC1> ip 192.168.20.10 255.255.255.0 192.168.20.1
Checking for duplicate address...
PC1 : 192.168.20.10 255.255.255.0 gateway 192.168.20.1

PC1> ping 192.168.20.1

84 bytes from 192.168.20.1 icmp_seq=1 ttl=64 time=0.888 ms
84 bytes from 192.168.20.1 icmp_seq=2 ttl=64 time=0.562 ms
84 bytes from 192.168.20.1 icmp_seq=3 ttl=64 time=0.405 ms
^C

```

## PC4 Network Configuration

```

PC4> ip 192.168.40.10 255.255.255.0 192.168.40.1
Checking for duplicate address...
PC4 : 192.168.40.10 255.255.255.0 gateway 192.168.40.1

PC4> ping 192.168.40.1

84 bytes from 192.168.40.1 icmp_seq=1 ttl=64 time=9.204 ms
84 bytes from 192.168.40.1 icmp_seq=2 ttl=64 time=0.437 ms
^C


```

This is good.  Each customer PC can reach their configured gateway, so basic LAN connectivity is in place.  Now, the final test is to confirm that PC2 
and PC3 can communicate with each other, but not with PC1 or PC4.  And vice versa the other direction.  We want PC1 and PC4 to be able to talk, but have no way of communicating
with PC2 or PC3.  

Let's test from each PC.

## PC1 Validation

```
# pinging PC4 is successful, pings to PC2 or PC3 return an ICMP Destination Unreachable message from PE1 since no route exists in the Customer A routing-instance

PC1> ping 192.168.40.10

84 bytes from 192.168.40.10 icmp_seq=1 ttl=61 time=3.273 ms
84 bytes from 192.168.40.10 icmp_seq=2 ttl=61 time=2.208 ms
84 bytes from 192.168.40.10 icmp_seq=3 ttl=61 time=2.542 ms
^C
PC1> ping 192.168.30.10

*192.168.20.1 icmp_seq=1 ttl=255 time=0.786 ms (ICMP type:3, code:0, Destination network unreachable)
*192.168.20.1 icmp_seq=2 ttl=255 time=0.582 ms (ICMP type:3, code:0, Destination network unreachable)
^C
PC1> ping 192.168.10.10

*192.168.20.1 icmp_seq=1 ttl=255 time=0.621 ms (ICMP type:3, code:0, Destination network unreachable)
*192.168.20.1 icmp_seq=2 ttl=255 time=0.646 ms (ICMP type:3, code:0, Destination network unreachable)
*192.168.20.1 icmp_seq=3 ttl=255 time=0.674 ms (ICMP type:3, code:0, Destination network unreachable)


```

## PC2 Validation

```

PC2> ping 192.168.30.10

84 bytes from 192.168.30.10 icmp_seq=1 ttl=61 time=2.911 ms
84 bytes from 192.168.30.10 icmp_seq=2 ttl=61 time=2.202 ms
84 bytes from 192.168.30.10 icmp_seq=3 ttl=61 time=2.129 ms
^C
PC2> ping 192.168.20.10

*192.168.10.1 icmp_seq=1 ttl=255 time=7.280 ms (ICMP type:3, code:0, Destination network unreachable)
*192.168.10.1 icmp_seq=2 ttl=255 time=0.553 ms (ICMP type:3, code:0, Destination network unreachable)
*192.168.10.1 icmp_seq=3 ttl=255 time=0.690 ms (ICMP type:3, code:0, Destination network unreachable)
^C
PC2> ping 192.168.40.10

*192.168.10.1 icmp_seq=1 ttl=255 time=0.623 ms (ICMP type:3, code:0, Destination network unreachable)
*192.168.10.1 icmp_seq=2 ttl=255 time=0.742 ms (ICMP type:3, code:0, Destination network unreachable)
*192.168.10.1 icmp_seq=3 ttl=255 time=0.586 ms (ICMP type:3, code:0, Destination network unreachable)
^C


```
## PC3 Validation

```

PC3> ping 192.168.10.10

84 bytes from 192.168.10.10 icmp_seq=1 ttl=61 time=2.159 ms
84 bytes from 192.168.10.10 icmp_seq=2 ttl=61 time=1.833 ms
84 bytes from 192.168.10.10 icmp_seq=3 ttl=61 time=3.547 ms
^C
PC3> ping 192.168.20.10

*192.168.30.1 icmp_seq=1 ttl=255 time=0.732 ms (ICMP type:3, code:0, Destination network unreachable)
*192.168.30.1 icmp_seq=2 ttl=255 time=0.531 ms (ICMP type:3, code:0, Destination network unreachable)
^C
PC3> ping 192.168.40.10

*192.168.30.1 icmp_seq=1 ttl=255 time=0.777 ms (ICMP type:3, code:0, Destination network unreachable)
*192.168.30.1 icmp_seq=2 ttl=255 time=0.524 ms (ICMP type:3, code:0, Destination network unreachable)
^C

```

## PC4 Validation




```

PC4> ping 192.168.20.10

84 bytes from 192.168.20.10 icmp_seq=1 ttl=61 time=2.855 ms
84 bytes from 192.168.20.10 icmp_seq=2 ttl=61 time=2.544 ms
^C
PC4> ping 192.168.10.10

*192.168.40.1 icmp_seq=1 ttl=255 time=0.536 ms (ICMP type:3, code:0, Destination network unreachable)
*192.168.40.1 icmp_seq=2 ttl=255 time=0.588 ms (ICMP type:3, code:0, Destination network unreachable)
^C
PC4> ping 192.168.30.10

*192.168.40.1 icmp_seq=1 ttl=255 time=0.807 ms (ICMP type:3, code:0, Destination network unreachable)
*192.168.40.1 icmp_seq=2 ttl=255 time=0.529 ms (ICMP type:3, code:0, Destination network unreachable)
^C
```

The same story on the rest of our PCs.  So our (simple) network is up and running.  Now how did we get there?

## OSPF Validation

### PE1

From PE1, we have formed a neighborship with the P Router through Area 0.  Also by checking the OSPF Database, we see the Loopback addresses
from each of the three routers in our topology visible in PE1's Database.  The Loopbacks are Router LSAs originating from each router in Area Zero.
This is important for PE1 and PE3 to be able to have end to end connectivity to form their BGP peering through the loopback addresses.

PE1 then generates a route based off of the total metric that it can calculate based off of the LSA data in the OSPF Database.

The metric for the OSPF route is calculated based off of the sum of all "costs" of the interfaces in the path of the route. 
The cost is a function of the reference bandwidth divided by the interface bandwidth in order to scale down the metric.  
By default, Junos uses 100Mbps as the reference bandwidth, and every link in this small core setup is 1Gbps.  Therefore,
to calculate the cost you take 100 / 1000.  However, this gives us 0.1 and OSPF takes any cost calculated to be less than one and rounds up to one.

Therefore, the cost for all of our links in this lab is 1.  Since the loopback to PE2 is two hops away, the metric is 2.

```

jsullivan@EVO-Sullivan-PE1> show ospf neighbor detail 
Address          Interface              State           ID               Pri  Dead
10.0.0.1         ge-0/0/4.0             Full            192.168.1.3      128    31
  Area 0.0.0.0, opt 0x52, DR 10.0.0.1, BDR 10.0.0.2
  Up 00:21:40, adjacent 00:20:55

jsullivan@EVO-Sullivan-PE1> show ospf database              

    OSPF database, Area 0.0.0.0
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router  *192.168.1.1      192.168.1.1      0x80000003  1395  0x22 0xf8ce  48
Router   192.168.1.2      192.168.1.2      0x80000003  1414  0x22 0x5f5d  48
Router   192.168.1.3      192.168.1.3      0x80000004  1396  0x22 0x4350  60
Network  10.0.0.1         192.168.1.3      0x80000001  1396  0x22 0xd813  32
Network  10.0.0.5         192.168.1.3      0x80000002   246  0x22 0xbc29  32

jsullivan@EVO-Sullivan-PE1> show route protocol ospf 192.168.1.2 

inet.0: 8 destinations, 8 routes (8 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

192.168.1.2/32     *[OSPF/10] 00:25:27, metric 2
                    >  to 10.0.0.1 via ge-0/0/4.0

inet.3: 2 destinations, 2 routes (2 active, 0 holddown, 0 hidden)



```

### P Router

```

jsullivan@EVO-Sullivan-P-Router> show ospf database 

    OSPF database, Area 0.0.0.0
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router   192.168.1.1      192.168.1.1      0x80000003  2562  0x22 0xf8ce  48
Router   192.168.1.2      192.168.1.2      0x80000003  2579  0x22 0x5f5d  48
Router  *192.168.1.3      192.168.1.3      0x80000005   486  0x22 0x4151  60
Network *10.0.0.1         192.168.1.3      0x80000001  2561  0x22 0xd813  32
Network *10.0.0.5         192.168.1.3      0x80000002  1411  0x22 0xbc29  32

jsullivan@EVO-Sullivan-P-Router> show ospf neighbor 
Address          Interface              State           ID               Pri  Dead
10.0.0.2         ge-0/0/4.0             Full            192.168.1.1      128    36
10.0.0.6         ge-0/0/5.0             Full            192.168.1.2      128    34

jsullivan@EVO-Sullivan-P-Router> show route protocol ospf 

inet.0: 9 destinations, 9 routes (9 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

192.168.1.1/32     *[OSPF/10] 00:42:48, metric 1
                    >  to 10.0.0.2 via ge-0/0/4.0
192.168.1.2/32     *[OSPF/10] 00:43:05, metric 1
                    >  to 10.0.0.6 via ge-0/0/5.0
224.0.0.5/32       *[OSPF/10] 00:44:07, metric 1
                       MultiRecv



```

### PE2 

PE2 will essentially be a mirror of what we saw on PE1.

```

jsullivan@EVO-Sullivan-PE2> show ospf neighbor 
Address          Interface              State           ID               Pri  Dead
10.0.0.5         ge-0/0/4.0             Full            192.168.1.3      128    38

jsullivan@EVO-Sullivan-PE2> show ospf database 

    OSPF database, Area 0.0.0.0
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router   192.168.1.1      192.168.1.1      0x80000003  2635  0x22 0xf8ce  48
Router  *192.168.1.2      192.168.1.2      0x80000003  2649  0x22 0x5f5d  48
Router   192.168.1.3      192.168.1.3      0x80000005   559  0x22 0x4151  60
Network  10.0.0.1         192.168.1.3      0x80000001  2634  0x22 0xd813  32
Network  10.0.0.5         192.168.1.3      0x80000002  1483  0x22 0xbc29  32

jsullivan@EVO-Sullivan-PE2> show route protocol ospf 

inet.0: 8 destinations, 8 routes (8 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.0.0.0/30        *[OSPF/10] 00:44:08, metric 2
                    >  to 10.0.0.5 via ge-0/0/4.0
192.168.1.1/32     *[OSPF/10] 00:43:50, metric 2
                    >  to 10.0.0.5 via ge-0/0/4.0
192.168.1.3/32     *[OSPF/10] 00:44:08, metric 1
                    >  to 10.0.0.5 via ge-0/0/4.0
224.0.0.5/32       *[OSPF/10] 00:45:11, metric 1
                       MultiRecv

```


## BGP Validation

Since the OSPF underlay routes to the loopbacks have been learned, let's see if BGP is up between PE1 and PE2.

### PE1

```
jsullivan@EVO-Sullivan-PE1> show bgp summary                                
Threading mode: BGP I/O
Default eBGP mode: advertise - accept, receive - accept
Groups: 1 Peers: 1 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
bgp.l3vpn.0          
                       2          2          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
192.168.1.2           100        107        106       0       0       45:37 Establ
  bgp.l3vpn.0: 2/2/2/0
  Customer-A.inet.0: 1/1/1/0
  Customer-B.inet.0: 1/1/1/0

jsullivan@EVO-Sullivan-PE1> show bgp neighbor 192.168.1.2 | match NLRI    
  NLRI for restart configured on peer: inet-vpn-unicast
  NLRI advertised by peer: inet-vpn-unicast
  NLRI for this session: inet-vpn-unicast
  NLRI that restart is negotiated for: inet-vpn-unicast
  NLRI of received end-of-rib markers: inet-vpn-unicast
  NLRI of all end-of-rib markers sent: inet-vpn-unicast
  NLRI(s) enabled for color nexthop resolution: inet-vpn-unicast


jsullivan@EVO-Sullivan-PE1> show route receive-protocol bgp 192.168.1.2    

inet.0: 8 destinations, 8 routes (8 active, 0 holddown, 0 hidden)

inet.3: 2 destinations, 2 routes (2 active, 0 holddown, 0 hidden)

Customer-A.inet.0: 3 destinations, 3 routes (3 active, 0 holddown, 0 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
* 192.168.30.0/24         192.168.1.2                  100        I

Customer-B.inet.0: 3 destinations, 3 routes (3 active, 0 holddown, 0 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
* 192.168.40.0/24         192.168.1.2                  100        I

mpls.0: 11 destinations, 11 routes (11 active, 0 holddown, 0 hidden)

bgp.l3vpn.0: 2 destinations, 2 routes (2 active, 0 holddown, 0 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
  192.168.1.2:100:192.168.30.0/24                    
*                         192.168.1.2                  100        I
  192.168.1.2:200:192.168.40.0/24                    
*                         192.168.1.2                  100        I

jsullivan@EVO-Sullivan-PE1> show route advertising-protocol bgp 192.168.1.2   

Customer-A.inet.0: 3 destinations, 3 routes (3 active, 0 holddown, 0 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
* 192.168.10.0/24         Self                         100        I

Customer-B.inet.0: 3 destinations, 3 routes (3 active, 0 holddown, 0 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
* 192.168.20.0/24         Self                         100        I


```

### PE2

```

jsullivan@EVO-Sullivan-PE2> show bgp summary 
Threading mode: BGP I/O
Default eBGP mode: advertise - accept, receive - accept
Groups: 1 Peers: 1 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
bgp.l3vpn.0          
                       2          2          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
192.168.1.1             100        114        113       0       0       48:51 Establ
  bgp.l3vpn.0: 2/2/2/0
  Customer-A.inet.0: 1/1/1/0
  Customer-B.inet.0: 1/1/1/0

jsullivan@EVO-Sullivan-PE2> show bgp neighbor 192.168.1.1 | match NLRI 
  NLRI for restart configured on peer: inet-vpn-unicast
  NLRI advertised by peer: inet-vpn-unicast
  NLRI for this session: inet-vpn-unicast
  NLRI that restart is negotiated for: inet-vpn-unicast
  NLRI of received end-of-rib markers: inet-vpn-unicast
  NLRI of all end-of-rib markers sent: inet-vpn-unicast
  NLRI(s) enabled for color nexthop resolution: inet-vpn-unicast

jsullivan@EVO-Sullivan-PE2> show route receive-protocol bgp 192.168.1.1 

inet.0: 8 destinations, 8 routes (8 active, 0 holddown, 0 hidden)

inet.3: 2 destinations, 2 routes (2 active, 0 holddown, 0 hidden)

Customer-A.inet.0: 3 destinations, 3 routes (3 active, 0 holddown, 0 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
* 192.168.10.0/24         192.168.1.1                  100        I

Customer-B.inet.0: 3 destinations, 3 routes (3 active, 0 holddown, 0 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
* 192.168.20.0/24         192.168.1.1                  100        I

mpls.0: 11 destinations, 11 routes (11 active, 0 holddown, 0 hidden)

bgp.l3vpn.0: 2 destinations, 2 routes (2 active, 0 holddown, 0 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
  192.168.1.1:100:192.168.10.0/24                    
*                         192.168.1.1                  100        I
  192.168.1.1:200:192.168.20.0/24                    
*                         192.168.1.1                  100        I

inet6.0: 2 destinations, 2 routes (2 active, 0 holddown, 0 hidden)
                                        
jsullivan@EVO-Sullivan-PE2> show route advertising-protocol bgp 192.168.1.1                      

Customer-A.inet.0: 3 destinations, 3 routes (3 active, 0 holddown, 0 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
* 192.168.30.0/24         Self                         100        I

Customer-B.inet.0: 3 destinations, 3 routes (3 active, 0 holddown, 0 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
* 192.168.40.0/24         Self                         100        I




```

As we can see, BGP is up and the proper routes are being exchanged in the Customer A and B VRFs.  We also see inet-vpn-unicast NLRI capability negotiated between
the two.  At a high level we can see that the routes are properly installed in the correct VRFs. 

In the Labels_and_Tables.md document, we'll take a deeper dive into how the LDP bindings, MPLS labels, and different routing-instances all interact in order to get this 
communication established.

