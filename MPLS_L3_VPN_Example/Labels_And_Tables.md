# Forwarding Labels and Tables Analysis

<img width="783" alt="Screenshot 2024-10-28 at 9 59 02 PM" src="https://github.com/user-attachments/assets/89e04a95-06ff-4b96-8abd-bf3f56be92e5">


We've validated that end to end connectivity works between our end hosts.  And also validated that inter-VRF isolation is working as expected as evidenced by pings
failing between hosts on VRF A and B.  Next, we'll take a deeper dive into how this traffic segmentation works, and how the routers are able to differentiate the traffic 
between the two different customers.

After we configure the PE and P routers we see that several new route tables have been generated in the Junos dataplane.  For example below, we see the default inet.0 table but
also:



```
jsullivan@EVO-Sullivan-PE1> show route table ?                    
Possible completions:
  <table>              Name of routing table
  Customer-A.inet.0    
  Customer-A.inet6.0   
  Customer-B.inet.0    
  Customer-B.inet6.0   
  bgp.l3vpn.0          
  inet.0               
  inet.3               
  inet6.0              
  mpls.0 

```

## MPLS.0 Table

We'll start off with the mpls.0 table.  This table is created when any interface is enabled with MPLS.  On PE1, this is done under ge-0/0/4. Let's take a look at ours looks like on PE1.

```

jsullivan@EVO-Sullivan-PE1> show route table mpls.0 

mpls.0: 11 destinations, 11 routes (11 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0                  *[MPLS/0] 01:01:40, metric 1
                       to table inet.0
0(S=0)             *[MPLS/0] 01:01:40, metric 1
                       to table mpls.0
1                  *[MPLS/0] 01:01:40, metric 1
                       Receive
2                  *[MPLS/0] 01:01:40, metric 1
                       to table inet6.0
2(S=0)             *[MPLS/0] 01:01:40, metric 1
                       to table mpls.0
13                 *[MPLS/0] 01:01:40, metric 1
                       Receive
16                 *[VPN/0] 01:01:40
                    >  via lsi.0 (Customer-A), Pop      
17                 *[VPN/0] 01:01:40
                    >  via lsi.1 (Customer-B), Pop      
299840             *[LDP/9] 00:11:34, metric 1
                    >  to 10.0.0.1 via ge-0/0/4.0, Pop      
299840(S=0)        *[LDP/9] 00:11:34, metric 1
                    >  to 10.0.0.1 via ge-0/0/4.0, Pop      
299856             *[LDP/9] 00:11:34, metric 1
                    >  to 10.0.0.1 via ge-0/0/4.0, Swap 299776

```

The first label we see is zero and the action tells us to look at inet.0. As you may or may not know, this is also referred as an "Explicit Null" label.

TO BE CONTINUED
