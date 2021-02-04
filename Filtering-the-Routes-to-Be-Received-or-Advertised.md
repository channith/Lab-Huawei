<h2>Filtering the Routes to Be Received or Advertised</h2>

![image](https://user-images.githubusercontent.com/63696723/106857890-67d48880-66f3-11eb-8a9c-64302d760ae0.png)

<h3>Configuration Roadmap</h3>

The configuration roadmap is as follows:

- Configure a routing policy on SwitchA and apply the routing policy during route advertisement. When routes are advertised, the routing policy allows SwitchA to provide routes from network segments 172.16.17.0/24, 172.16.18.0/24, and 172.16.19.0/24 for SwitchB, and allows devices on the OSPF network to access these three network segments.

- Configure a routing policy on SwitchC and apply the routing policy during route importing. When routes are imported, the routing policy allows SwitchC to receive only the routes from the network segment 172.16.18.0/24 and access this network segment.

