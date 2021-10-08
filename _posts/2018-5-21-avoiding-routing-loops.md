---
title: 'Avoiding Route Distribution Loops'
---

I have a scenario where I am deploying an MPLS network to replace  multiple L3 networks that I currently support.  To provide continuity between a legacy L3 network and the MPLS VPRN, I need to share routes between the two network and appropriately redistribute.  All the while, avoiding a route distribution loop.  

Consider the following diagram as a mockup of my lab:

<p align="center">
  <img src="https://cpajr.com/assets/2018-5-21-diagram.png">
</p>

### The Problem

Based on the above diagram, I need to avoid a route distribution loop which can result in suboptimal routing.  The following diagram can better reflect the potential loop:

<p align="center">
  <img src="https://cpajr.com/assets/2018-05-21-diagram2.png">
</p>

1. Routes received via EIGRP for the legacy network
2. Routes from EIGRP are redistributed into SR1 via BGP
3. Those VPRN specific routes received via BGP are redistributed into MP-BGP
4. SR2 will then redistribute MP-BGP routes into BGP to IOS2
5. IOS2 will then redistribute BGP routes into EIGRP, completing the distribution loop

To avoid this scenario I will tag routes received on both sides of BGP and then create policy to deny redistribution:

<p align="center">
  <img src="https://cpajr.com/assets/2018-05-21-diagram3.png">
</p>

### Configuration

I have a mixture of two different vendors and, as expected, I needed to figure out the nuances between them.  The configuration from IOS1:
```
interface GigabitEthernet1/0/24
 no switchport
 ip address 10.202.0.1 255.255.255.254
 bfd interval 50 min_rx 50 multiplier 3
!
router eigrp 1000
 network 10.0.0.0
 redistribute connected
 redistribute static
 redistribute bgp 650002 metric 100 1 255 1 1500 route-map bgp-to-eigrp
 passive-interface default
 no passive-interface GigabitEthernet1/1/1
!
router bgp 65001
 bgp log-neighbor-changes
 bgp redistribute-internal
 network 10.0.0.0
 redistribute connected
 redistribute static
 redistribute eigrp 1000 route-map eigrp-to-bgp
 neighbor 10.202.0.0 remote-as 65001
 neighbor 10.202.0.0 password <password>
 neighbor 10.202.0.0 fall-over bfd
 neighbor 10.202.0.0 next-hop-self
!
route-map bgp-to-eigrp permit 10
 set tag 1001
!
route-map eigrp-to-bgp deny 10
 match tag 1001
!
route-map eigrp-to-bgp permit 20
!
```
For SR1, I have included the VPRN configuration:
```
router-id 172.16.1.1
autonomous-system 65001
interface "to-cisco" create
    address 10.202.0.0/31
    bfd 50 receive 50 multiplier 3 type np
    sap 1/1/7 create
    exit
exit
interface "loopback" create
    address 10.204.250.3/32
    loopback
exit
bgp
    bfd-enable
    group "cisco"
        min-route-advertisement 1
        neighbor 10.202.0.1
			authentication-key <password>
			next-hop-self
			import "add-SoO"
			export "noLoop-ToCE"
			peer-as 65001
        exit
    exit
    no shutdown
exit
no shutdown
```
As you can notice, I have also configured BFD between the two routers so I can have a smaller convergence time.  

For IOS2, the following is the configuration:
```
interface GigabitEthernet1/0/24
 no switchport
 ip address 10.202.0.3 255.255.255.254
 bfd interval 50 min_rx 50 multiplier 3
!
router eigrp 1000
 network 10.0.0.0
 redistribute connected
 redistribute static
 redistribute bgp 65001 metric 100 1 255 1 1500 route-map bgp-to-eigrp
 passive-interface default
 no passive-interface GigabitEthernet1/1/1
!
router bgp 65001
 bgp log-neighbor-changes
 bgp redistribute-internal
 network 10.0.0.0
 redistribute connected
 redistribute static
 redistribute eigrp 1000 route-map eigrp-to-bgp
 neighbor 10.202.0.2 remote-as 65001
 neighbor 10.202.0.2 password <password>
 neighbor 10.202.0.2 fall-over bfd
 neighbor 10.202.0.2 next-hop-self
!
route-map bgp-to-eigrp permit 10
 set tag 1001
!
route-map eigrp-to-bgp deny 10
 match tag 1001
!
route-map eigrp-to-bgp permit 20
!
```
For SR2, the following is the configuration:
```
router-id 172.16.1.2
autonomous-system 65001
interface "to-cisco" create
    address 10.202.0.2/31
    bfd 50 receive 50 multiplier 3 type np
    sap 1/1/7 create
    exit
exit
interface "loopback" create
    address 10.204.250.4/32
    loopback
exit
bgp
    bfd-enable
    group "cisco"
        min-route-advertisement 1
        neighbor 10.202.0.3
			authentication-key <password>
			next-hop-self
			import "add-SoO"
			export "noLoop-ToCE"
			peer-as 65001
        exit
    exit
    no shutdown
exit
no shutdown
```
For both SR nodes, I have used some policy statements:
```
router
    policy-options
        begin
        policy-statement "add-SoO" 
            entry 10 
                action accept 
                    community add "SoO" 
                exit 
            exit 
        exit 
        policy-statement "noLoop-ToCE" 
            entry 10 
                from  
                    protocol bgp-vpn 
                    community "SoO" 
                exit 
                action reject 
            exit 		 
            entry 20 
                from 
                    protocol bgp-vpn 
                exit 
                action accept 
            exit  
        exit  
```
### Lessons Learned
1. **BFD**: I learned that BFD will not intiate a session until it is tied to a protocol.  In my case, I configured BFD on the interfaces, but the session wouldn't establish.  It wasn't until I tied it to BGP did the session begin.  
2. **BGP Redistribute Internal**: I was struggling at one point to have BGP redistribute routes into EIGRP; it just would not work.  Finally, I found a command for `bgp redistribute-internal`.  Apparently, by default for Cisco, it will not redistribute iBGP routes to another protocol.  
3. **Policy Statements on Nokia**: I had to give careful attention to the policy statement on the Nokia SR routers.  I kept missing statement to redistribute local or connected routes.  
4. **Route Filtering on Cisco**: with the route-map I created for deny tagged routes, I had not originally included a permit statement (see the `route-map eigrp-to-bgp`).  By default, Cisco places an implied deny with its route-maps; I needed to add a permit to allow routes, which were not tagged, to be distributed into BGP.  
