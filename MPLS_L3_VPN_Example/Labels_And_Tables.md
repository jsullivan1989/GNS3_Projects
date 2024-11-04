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

UNDER CONSTRUCTION
