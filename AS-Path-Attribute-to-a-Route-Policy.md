<h2>Applying the AS-Path Attribute to a Route-Policy</h2>

---------------------------------------------------------

<h3>1. Networking Requirements</h3>

  As shown in Figure below, four L3 switches belong to different ASs and establish eBGP connections.
  
![Applying the AS-Path Attribute to a Route-Policy](https://user-images.githubusercontent.com/63696723/109672798-f4198480-7ba7-11eb-8a9e-ff25fb3d54f8.png)

When LSW2 sends route (192.168.21.0/24) to LSW1, the AS path attribute needs to be changed so that
route from LSW1 to 192.168.21.0/24 is changed.

<h3>2. Procedure</h3>

<h4>2.1. Basic configuration</h4>

- LSW1
```
sys
sysn LSW1

vlan batch 1 12 13

int vlan 1
 ip address 192.168.1.1 24
 quit
int vlan 12
 ip address 192.168.12.1 24
 quit
int vlan 13
 ip address 192.168.13.1 24
 quit
 
int gi0/0/1
 port link-type trunk
 port trunk allow-pass vlan 1
 quit
 
int gi0/0/2
 port link-type trunk
 undo port trunk allow-pass vlan 1
 port trunk allow-pass vlan 13
 stp disable
 quit

int gi0/0/3
 port link-type trunk
 undo port trunk allow-pass vlan 1
 port trunk allow-pass vlan 12
 stp disable
 quit
```

- LSW2
```
sys
sysn LSW2

vlan batch 12 24 21

int vlan 21
 ip address 192.168.21.1 24
 quit
int vlan 12
 ip address 192.168.12.2 24
 quit
int vlan 24
 ip address 192.168.24.2 24
 quit
 
int loopback 0
 ip add 2.2.2.2 32
 quit
int gi0/0/3
 port link-type trunk
 undo port trunk allow-pass vlan 1
 port trunk allow-pass vlan 21
 stp disable
 quit
 
int gi0/0/2
 port link-type trunk
 undo port trunk allow-pass vlan 1
 port trunk allow-pass vlan 12
 stp disable
 quit
 
int gi0/0/1
 port link-type trunk
 undo port trunk allow-pass vlan 1
 port trunk allow-pass vlan 24
 stp disable
 quit
 
ip route-static 22.2.2.0 24 null 0
 
```

- LSW3
```
sys
sysn LSW3

vlan batch 13 34

int vlan 13
 ip address 192.168.13.3 24
 quit
int vlan 34
 ip address 192.168.34.3 24
 quit
 
int gi0/0/1
 port link-type trunk
 undo port trunk allow-pass vlan 1
 port trunk allow-pass vlan 13
 stp disable
 quit
 
int gi0/0/2
 port link-type trunk
 undo port trunk allow-pass vlan 1
 port trunk allow-pass vlan 34
 stp disable
 quit
```

- LSW4
```
sys
sysn LSW4

vlan batch 34 24

int vlan 34
 ip address 192.168.34.4 24
 quit
int vlan 24
 ip address 192.168.24.4 24
 quit
 
int gi0/0/1
 port link-type trunk
 undo port trunk allow-pass vlan 1
 port trunk allow-pass vlan 34
 stp disable
 quit
 
int gi0/0/2
 port link-type trunk
 undo port trunk allow-pass vlan 1
 port trunk allow-pass vlan 24
 stp disable
 quit
```

<h4>2.2. BGP configuration</h4>

- LSW1
```
bgp 100
 router-id 1.1.1.1
 peer 192.168.12.2 as-number 200
 peer 192.168.13.3 as-number 300
 
 ipv4-family unicast
  network 192.168.1.0 24
  quit
 quit
```

- LSW2
```
bgp 200
 router-id 2.2.2.2
 peer 192.168.12.1 as-number 100
 peer 192.168.24.4 as-number 400
 
 ipv4-family unicast
  network 192.168.21.0 24
  network 2.2.2.2 32
  network 22.2.2.0 24
  quit
 quit
```

- LSW3
```
bgp 300
 router-id 3.3.3.3
 peer 192.168.34.4 as-number 400
 peer 192.168.13.1 as-number 100
 
```

- LSW4
```
bgp 400
 router-id 4.4.4.4
 peer 192.168.34.3 as-number 300
 peer 192.168.24.2 as-number 200
 
```

- Check eBGPs establish

![image](https://user-images.githubusercontent.com/63696723/109679802-7efd7d80-7bae-11eb-8a4a-84f81b3e02cd.png)

<h3>3. Change AS-Path attribute</h3>

Before a route-policy is applied to LSW2, run the **display bgp routing-table** command on LSW1.

![image](https://user-images.githubusercontent.com/63696723/109685978-64c69e00-7bb4-11eb-940c-4364c31ffde5.png)

You can see a BGP route destined for 192.168.21.0/24 (should has 2 BGP routes). 
A route with next-hop address 192.168.12.2 has AS-Path 200 (one more next-hop address 192.168.13.3 has AS-Path
300 400 200).
Then run the display ip routing-table command. 
You can see that the route with next-hop address 192.168.12.2 is preferred.

![image](https://user-images.githubusercontent.com/63696723/109687254-a572e700-7bb5-11eb-8abe-95e6c15a0fa7.png)


After a route-policy is applied to LSW2, run the **display bgp routing-table** command on LSW1.

```
#LSW2
bgp 200
 router-id 2.2.2.2
 peer 192.168.12.1 as-number 100
 peer 192.168.24.4 as-number 400
 #
 ipv4-family unicast
  undo synchronization
  network 2.2.2.2 255.255.255.255 //Configure BGP to advertise local routes.
  network 22.2.2.0 255.255.255.0
  network 192.168.21.0
  peer 192.168.12.1 enable
  peer 192.168.12.1 route-policy Route-200-export export //Apply a route-policy to the advertised routes.
  peer 192.168.24.4 enable
#
route-policy Route-200-export permit node 10 //Create a route-policy.
 if-match ip-prefix Route-21
 apply as-path 200 200 200 additive //Add AS number 200 200 200 to the AS-Path list.
#
route-policy Route-200-export permit node 1000 // Other prefixes are advertise normal
#
ip ip-prefix Route-21 index 10 permit 192.168.21.0 24
#
ip route-static 22.2.2.0 255.255.255.0 NULL0
#
```

Result:

![image](https://user-images.githubusercontent.com/63696723/109688349-c25bea00-7bb6-11eb-859e-8b78307ee3d0.png)

![image](https://user-images.githubusercontent.com/63696723/109689264-b4f32f80-7bb7-11eb-979c-6dc6d0c65d99.png)

and here in routing table:

![image](https://user-images.githubusercontent.com/63696723/109688498-e9b2b700-7bb6-11eb-98d8-780109b137f6.png)

^-^ G9!
