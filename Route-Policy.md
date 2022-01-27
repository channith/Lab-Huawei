<h2>Route Policy</h2>



<h3>1. Definition</h3>

- Routing policy is a policy that controls routes using a series tools or methods. 
This type of policy will affect route generation, advertisement, 
and selection and so affect the packet forwarding path. 

- These tools include ACL, route-policy, ip-prefix, and filter-policy, 
and these methods include filtering routes and setting attributes for routes.

<h4>1.1  Invoking Between Tools Used in Routing Policy</h4>

![image](https://user-images.githubusercontent.com/63696723/151126340-644444e0-1cbd-4bb9-a728-1f99c4049d0f.png)

- Condition tool: captures required routes
- Policy tool: performs an action on the captured routes, for example, permit, deny, and modify attributes.
- Invoking tool: applies a routing policy to a specific routing protocol to make the routing policy to take effect.

Among invoking tools, filter-policy and peer have policy tools and can directly invoke condition tools. 
Other invoking tools must use route-policy to invoke condition tools.

<h3>2. LAB</h3>

<h4>2.1. Diagram</h4>

![image](https://user-images.githubusercontent.com/63696723/151276236-b33a66a0-fd50-4999-8311-aa44f542cbeb.png)


- OSPF

```
LSW1#
ospf 2106 router-id 1.1.1.1
 area 0.0.0.1

interface Vlanif10
 ip address 192.168.12.1 255.255.255.0
 ospf enable 2106 area 0.0.0.1

interface LoopBack0
 ip address 172.16.1.1 255.255.255.0


 
 LSW2#

interface Vlanif10
 ip address 192.168.12.2 255.255.255.0
 ospf enable 22 area 0.0.0.1

interface LoopBack0
 ip address 2.2.2.2 255.255.255.0
 ospf network-type broadcast
 ospf enable 22 area 0.0.0.1

```

<h4>2.2. Filter routes, using ACL and route-policy</h4>

Suppose we have this RIP in LSW2:
```
[LSW1]dis ip routing-table 
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Tables: Public
         Destinations : 10       Routes : 10       

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

        2.2.2.0/24  OSPF    10   1           D   192.168.12.2    Vlanif10
      127.0.0.0/8   Direct  0    0           D   127.0.0.1       InLoopBack0
      127.0.0.1/32  Direct  0    0           D   127.0.0.1       InLoopBack0
     172.16.1.0/24  Direct  0    0           D   172.16.1.1      LoopBack0
     172.16.1.1/32  Direct  0    0           D   127.0.0.1       LoopBack0
    172.16.11.0/24  Static  60   0           D   0.0.0.0         NULL0
    172.16.12.0/24  Static  60   0           D   0.0.0.0         NULL0
    172.16.13.0/24  Static  60   0           D   0.0.0.0         NULL0
   192.168.12.0/24  Direct  0    0           D   192.168.12.1    Vlanif10
   192.168.12.1/32  Direct  0    0           D   127.0.0.1       Vlanif10

[LSW1]
```

And we want to import 172.16.11.0/24, 172.16.12.0/24 from static route into OSPF.

- Access-list
```
acl name static2ospf 2005
 rule 10 permit source 172.16.11.0 0.0.0.255
 rule 15 permit source 172.16.12.0 0.0.0.255
```

- Route-policy
```
route-policy static2ospf permit node 10
 if-match acl static2ospf
```

- Apply into OSPF
```
ospf 2106 router-id 1.1.1.1
 import-route static route-policy static2ospf
 area 0.0.0.1
```

- Result:
```
<LSW2>dis ip rout
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Tables: Public
         Destinations : 10       Routes : 10       

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

        2.2.2.0/24  Direct  0    0           D   2.2.2.2         LoopBack0
        2.2.2.2/32  Direct  0    0           D   127.0.0.1       LoopBack0
      127.0.0.0/8   Direct  0    0           D   127.0.0.1       InLoopBack0
      127.0.0.1/32  Direct  0    0           D   127.0.0.1       InLoopBack0
    172.16.11.0/24  O_ASE   150  1           D   192.168.12.1    Vlanif10
    172.16.12.0/24  O_ASE   150  1           D   192.168.12.1    Vlanif10
   192.168.12.0/24  Direct  0    0           D   192.168.12.2    Vlanif10
   192.168.12.2/32  Direct  0    0           D   127.0.0.1       Vlanif10
   192.168.23.0/24  Direct  0    0           D   192.168.23.2    Vlanif20
   192.168.23.2/32  Direct  0    0           D   127.0.0.1       Vlanif20

<LSW2>
```

<h4>2.2. Filter routes, using IP prefix-list and filter-policy</h4>

Suppose on Switch LSW2 we want to import ISIS routes into OSPF network. But want the route to 172.30.1.0/24 on the OSPF network has 
a low preference and route to 172.30.2.0/24 has a tag, which makes it easy to reference by a routing policy.

```
LSW3#
isis 2601
 cost-style wide
 network-entity 49.0001.0030.0300.3003.00
 is-name LSW3

#
interface Vlanif20
 ip address 192.168.23.3 255.255.255.0
 isis enable 2601

#
interface GigabitEthernet0/0/1
 port link-type trunk
 undo port trunk allow-pass vlan 1
 port trunk allow-pass vlan 20
#
interface LoopBack0
 ip address 172.30.1.1 255.255.255.0
 isis enable 2601
#
interface LoopBack1
 ip address 172.30.2.1 255.255.255.0
 isis enable 2601
#
interface LoopBack2
 ip address 172.30.3.1 255.255.255.0
 isis enable 2601


LSW2#

#
acl number 2010
 rule 10 permit source 172.30.2.0 0.0.0.255
#

isis 2601
 cost-style wide
 network-entity 49.0001.0020.0200.2002.00
 is-name LSW2
#
interface Vlanif10
 ip address 192.168.12.2 255.255.255.0
 ospf enable 22 area 0.0.0.1
#
interface Vlanif20
 ip address 192.168.23.2 255.255.255.0
 isis enable 2601

#
interface GigabitEthernet0/0/1
 port link-type trunk
 undo port trunk allow-pass vlan 1
 port trunk allow-pass vlan 10
#
interface GigabitEthernet0/0/2
 port link-type trunk
 undo port trunk allow-pass vlan 1
 port trunk allow-pass vlan 20
#

interface LoopBack0
 ip address 2.2.2.2 255.255.255.0
 isis enable 2601
 ospf network-type broadcast
 ospf enable 22 area 0.0.0.1
#
ospf 22 router-id 2.2.2.2
 import-route isis 2601 route-policy isis2ospf
 area 0.0.0.1
#
route-policy isis2ospf permit node 10
 if-match ip-prefix isis2ospf
 apply cost 100
#
route-policy isis2ospf permit node 20
 if-match acl 2010
 apply tag 20
#
route-policy isis2ospf permit node 30
#
ip ip-prefix isis2ospf index 10 permit 172.30.1.0 24

```

- Result
```
<LSW1>dis ospf routing 

	 OSPF Process 2106 with Router ID 1.1.1.1
		  Routing Tables 

 Routing for Network 
 Destination        Cost  Type       NextHop         AdvRouter       Area
 192.168.12.0/24    1     Transit    192.168.12.1    1.1.1.1         0.0.0.1
 2.2.2.0/24         1     Stub       192.168.12.2    2.2.2.2         0.0.0.1

 Routing for ASEs
 Destination        Cost      Type       Tag         NextHop         AdvRouter
 172.30.1.0/24      100       Type2      1           192.168.12.2    2.2.2.2
 172.30.2.0/24      1         Type2      20          192.168.12.2    2.2.2.2
 172.30.3.0/24      1         Type2      1           192.168.12.2    2.2.2.2
 192.168.23.0/24    1         Type2      1           192.168.12.2    2.2.2.2

 Total Nets: 6  
 Intra Area: 2  Inter Area: 0  ASE: 4  NSSA: 0 

<LSW1>
<LSW1>
<LSW1>dis ip rout	
<LSW1>dis ip routing-table 
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Tables: Public
         Destinations : 14       Routes : 14       

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

        2.2.2.0/24  OSPF    10   1           D   192.168.12.2    Vlanif10
      127.0.0.0/8   Direct  0    0           D   127.0.0.1       InLoopBack0
      127.0.0.1/32  Direct  0    0           D   127.0.0.1       InLoopBack0
     172.16.1.0/24  Direct  0    0           D   172.16.1.1      LoopBack0
     172.16.1.1/32  Direct  0    0           D   127.0.0.1       LoopBack0
    172.16.11.0/24  Static  60   0           D   0.0.0.0         NULL0
    172.16.12.0/24  Static  60   0           D   0.0.0.0         NULL0
    172.16.13.0/24  Static  60   0           D   0.0.0.0         NULL0
     172.30.1.0/24  O_ASE   150  100         D   192.168.12.2    Vlanif10
     172.30.2.0/24  O_ASE   150  1           D   192.168.12.2    Vlanif10
     172.30.3.0/24  O_ASE   150  1           D   192.168.12.2    Vlanif10
   192.168.12.0/24  Direct  0    0           D   192.168.12.1    Vlanif10
   192.168.12.1/32  Direct  0    0           D   127.0.0.1       Vlanif10
   192.168.23.0/24  O_ASE   150  1           D   192.168.12.2    Vlanif10

<LSW1>
```


- Now on LSW1, we do not want to import route 172.30.3.0/24 into RIB of LSW1.

```
LSW1#
ip ip-prefix DENY index 5 deny 172.30.3.0 24 // to deny import route to 172.30.3.0/24
ip ip-prefix DENY index 10 permit 0.0.0.0 0 less-equal 32 // to allow all

ospf 2106 router-id 1.1.1.1
 filter-policy ip-prefix DENY import
 import-route static route-policy static2ospf
 area 0.0.0.1
#




<LSW1>dis ip rout
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Tables: Public
         Destinations : 13       Routes : 13       

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

        2.2.2.0/24  OSPF    10   1           D   192.168.12.2    Vlanif10
      127.0.0.0/8   Direct  0    0           D   127.0.0.1       InLoopBack0
      127.0.0.1/32  Direct  0    0           D   127.0.0.1       InLoopBack0
     172.16.1.0/24  Direct  0    0           D   172.16.1.1      LoopBack0
     172.16.1.1/32  Direct  0    0           D   127.0.0.1       LoopBack0
    172.16.11.0/24  Static  60   0           D   0.0.0.0         NULL0
    172.16.12.0/24  Static  60   0           D   0.0.0.0         NULL0
    172.16.13.0/24  Static  60   0           D   0.0.0.0         NULL0
     172.30.1.0/24  O_ASE   150  100         D   192.168.12.2    Vlanif10
     172.30.2.0/24  O_ASE   150  1           D   192.168.12.2    Vlanif10
   192.168.12.0/24  Direct  0    0           D   192.168.12.1    Vlanif10
   192.168.12.1/32  Direct  0    0           D   127.0.0.1       Vlanif10
   192.168.23.0/24  O_ASE   150  1           D   192.168.12.2    Vlanif10

<LSW1>dis ospf rou	
<LSW1>dis ospf routing 

	 OSPF Process 2106 with Router ID 1.1.1.1
		  Routing Tables 

 Routing for Network 
 Destination        Cost  Type       NextHop         AdvRouter       Area
 192.168.12.0/24    1     Transit    192.168.12.1    1.1.1.1         0.0.0.1
 2.2.2.0/24         1     Stub       192.168.12.2    2.2.2.2         0.0.0.1

 Routing for ASEs
 Destination        Cost      Type       Tag         NextHop         AdvRouter
 172.30.1.0/24      100       Type2      1           192.168.12.2    2.2.2.2
 172.30.2.0/24      1         Type2      20          192.168.12.2    2.2.2.2
 172.30.3.0/24      1         Type2      1           192.168.12.2    2.2.2.2
 192.168.23.0/24    1         Type2      1           192.168.12.2    2.2.2.2

 Total Nets: 6  
 Intra Area: 2  Inter Area: 0  ASE: 4  NSSA: 0 


	 OSPF Process 2601 with Router ID 172.16.1.1
		  Routing Tables 

 Total Nets: 0  
 Intra Area: 0  Inter Area: 0  ASE: 0  NSSA: 0 

<LSW1>
```

https://support.huawei.com/enterprise/en/doc/EDOC1000027467?section=j00f
https://medium.com/@wehua/huawei-s-series-switches-routing-policy-1-routing-policy-143f4c80b128
