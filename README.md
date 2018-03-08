# cldemo-evpn-centralized

This Github repository contains the configuration files necessary for setting up EVPN using Cumulus Linux and FRR on the Reference Topology.  It includes VXLAN/VLAN bridging with BGP EVPN Control Plane along with the VXLAN Routing using the Centralized Architecture.

This demo is equivalent to the Centralized Architecture Deployment scenario in the [Cumulus VXLAN Routing and EVPN whitepaper](https://cumulusnetworks.com/learn/web-scale-networking-resources/whitepapers/Cumulus-Networks-White-Paper-EVPN.pdf) 

The flatfiles in this repository will set up a BGP unnumbered underlay along with an EVPN overlay between the leafs, spines and exit router (i.e. border leaf).  The servers will have a basic IPv4 with bonding (MLAG) to the leafs.  Server01 and 03 are in one VLAN/VXLAN, and servers 02 and 04 are in a different VLAN/VXLAN.  VTEPS are configured on all leaf switches and the exit switch with all VNIs.  Exit01 and Exit02 are configured as the centralized VXLAN Routers and are running MLAG between them. The SVIs are configured on the Exit routers and  the VRR virtual address is advertised as the default gateway to the Leaf switches via the EVPN BGP extended community. 

The internet router is advertising a default route to the exit router.  The internet router has IP address 172.16.1.1 as a loopback address.


Quickstart: Run the demo
------------------------

Before running this demo, install VirtualBox and Vagrant. The currently supported versions of VirtualBox and Vagrant can be found on the [cldemo-vagrant](https://github.com/CumulusNetworks/cldemo-vagrant) page.  

    git clone https://github.com/cumulusnetworks/cldemo-vagrant
    cd cldemo-vagrant
    vagrant up oob-mgmt-server oob-mgmt-switch 
    vagrant up leaf01 leaf02 leaf03 leaf04 spine01 spine02 exit01 exit02 internet server01 server02 server03 server04
    vagrant ssh oob-mgmt-server
    sudo su - cumulus
    git clone https://github.com/CumulusNetworks/cldemo-evpn-centralized
    cd cldemo-evpn-centralized
    ansible-playbook run-demo.yml
    ssh server01
    ping 172.16.1.1 


Software in Use
------------------------
**Spines, Leafs, Exit and Internet:**
      Cumulus v3.5.0

**On Servers:**
Ubuntu 16.04


## Topology ##

This demo runs on a spine-leaf topology with four attached hosts. The ansible playbook run-demo.yml requires an out-of-band management network that provides access to eth0 on all of the in-band devices. 

![EVPN Demo Topology](https://github.com/CumulusNetworks/cldemo-evpn-centralized/blob/master/cldemo-evpn-centralized.png)

 
## Viewing the Results ##

-------
Observe the Default Gateway Community getting distributed with the type 2 routes to the default gateway:

    cumulus@leaf01:mgmt-vrf:~$ net show bgp l2vpn evpn route rd 10.0.0.42:3
    EVPN type-2 prefix: [2]:[ESI]:[EthTag]:[MAClen]:[MAC]
    EVPN type-3 prefix: [3]:[EthTag]:[IPlen]:[OrigIP]
    EVPN type-5 prefix: [5]:[ESI]:[EthTag]:[IPlen]:[IP]
    
    BGP routing table entry for 10.0.0.42:3:[2]:[0]:[0]:[48]:[44:38:39:00:00:27]:[32]:[10.2.4.12]
    Paths: (1 available, best #1)
      Advertised to non peer-group peers:
      spine02(swp52)
      Route [2]:[0]:[0]:[48]:[44:38:39:00:00:27]:[32]:[10.2.4.12] VNI 24
      65020 65042
        10.0.0.142 from spine02(swp52) (10.0.0.22)
          Origin IGP, localpref 100, valid, external, bestpath-from-AS 65020, best
          Extended Community: RT:65042:24 ET:8 Default Gateway
          AddPath ID: RX 0, TX 56
          Last update: Thu Mar  8 19:58:56 2018
    
    BGP routing table entry for 10.0.0.42:3:[2]:[0]:[0]:[48]:[44:38:39:00:00:27]:[128]:[fe80::4638:39ff:fe00:27]
    Paths: (1 available, best #1)
      Advertised to non peer-group peers:
      spine02(swp52)
      Route [2]:[0]:[0]:[48]:[44:38:39:00:00:27]:[128]:[fe80::4638:39ff:fe00:27] VNI 24
      65020 65042
        10.0.0.142 from spine02(swp52) (10.0.0.22)
          Origin IGP, localpref 100, valid, external, bestpath-from-AS 65020, best
          Extended Community: RT:65042:24 ET:8 Default Gateway
          AddPath ID: RX 0, TX 62
          Last update: Thu Mar  8 19:58:56 2018
    
    BGP routing table entry for 10.0.0.42:3:[2]:[0]:[0]:[48]:[44:39:39:ff:00:24]:[32]:[10.2.4.1]
    Paths: (1 available, best #1)
      Advertised to non peer-group peers:
      spine02(swp52)
      Route [2]:[0]:[0]:[48]:[44:39:39:ff:00:24]:[32]:[10.2.4.1] VNI 24
      65020 65042
        10.0.0.142 from spine02(swp52) (10.0.0.22)
          Origin IGP, localpref 100, valid, external, bestpath-from-AS 65020, best
          Extended Community: RT:65042:24 ET:8 **Default Gateway**
          AddPath ID: RX 0, TX 60
          Last update: Thu Mar  8 19:58:56 2018
    
    BGP routing table entry for 10.0.0.42:3:[2]:[0]:[0]:[48]:[44:39:39:ff:00:24]:[128]:[fe80::4639:39ff:feff:24]
    Paths: (1 available, best #1)
      Advertised to non peer-group peers:
      spine02(swp52)
      Route [2]:[0]:[0]:[48]:[44:39:39:ff:00:24]:[128]:[fe80::4639:39ff:feff:24] VNI 24
      65020 65042
        10.0.0.142 from spine02(swp52) (10.0.0.22)
          Origin IGP, localpref 100, valid, external, bestpath-from-AS 65020, best
          Extended Community: RT:65042:24 ET:8 Default Gateway
          AddPath ID: RX 0, TX 58
          Last update: Thu Mar  8 19:58:56 2018
    
    BGP routing table entry for 10.0.0.42:3:[3]:[0]:[32]:[10.0.0.142]
    Paths: (1 available, best #1)
      Advertised to non peer-group peers:
      spine02(swp52)
      Route [3]:[0]:[32]:[10.0.0.142]
      65020 65042
        10.0.0.142 from spine02(swp52) (10.0.0.22)
          Origin IGP, localpref 100, valid, external, bestpath-from-AS 65020, best
          Extended Community: RT:65042:24 ET:8
          AddPath ID: RX 0, TX 64
          Last update: Thu Mar  8 19:58:56 2018
    
    
    Displayed 5 prefixes (5 paths) with this RD

    
Look at the Routing table on a centralized router

    cumulus@exit01:mgmt-vrf:~$ net show route
    
    show ip route
    =============
    Codes: K - kernel route, C - connected, S - static, R - RIP,
           O - OSPF, I - IS-IS, B - BGP, P - PIM, E - EIGRP, N - NHRP,
           T - Table, v - VNC, V - VNC-Direct, A - Babel, D - SHARP,
           > - selected route, * - FIB route
    
    B>* 0.0.0.0/0 [20/0] via fe80::4638:39ff:fe00:7, swp44, 00:55:50
    B>* 10.0.0.11/32 [20/0] via fe80::4638:39ff:fe00:5b, swp52, 00:55:49
      *                     via fe80::4638:39ff:fe00:a, swp51, 00:55:49
    B>* 10.0.0.12/32 [20/0] via fe80::4638:39ff:fe00:5b, swp52, 00:55:49
      *                     via fe80::4638:39ff:fe00:a, swp51, 00:55:49
    B>* 10.0.0.13/32 [20/0] via fe80::4638:39ff:fe00:5b, swp52, 00:55:49
      *                     via fe80::4638:39ff:fe00:a, swp51, 00:55:49
    B>* 10.0.0.14/32 [20/0] via fe80::4638:39ff:fe00:5b, swp52, 00:55:49
      *                     via fe80::4638:39ff:fe00:a, swp51, 00:55:49
    B>* 10.0.0.21/32 [20/0] via fe80::4638:39ff:fe00:a, swp51, 00:55:49
    B>* 10.0.0.22/32 [20/0] via fe80::4638:39ff:fe00:5b, swp52, 00:55:49
    C>* 10.0.0.41/32 is directly connected, lo, 00:55:53
    B>* 10.0.0.42/32 [20/0] via fe80::4638:39ff:fe00:a, swp51, 00:55:49
      *                     via fe80::4638:39ff:fe00:5b, swp52, 00:55:49
    B>* 10.0.0.112/32 [20/0] via fe80::4638:39ff:fe00:5b, swp52, 00:55:49
      *                      via fe80::4638:39ff:fe00:a, swp51, 00:55:49
    B>* 10.0.0.134/32 [20/0] via fe80::4638:39ff:fe00:5b, swp52, 00:55:49
      *                      via fe80::4638:39ff:fe00:a, swp51, 00:55:49
    C>* 10.0.0.142/32 is directly connected, lo, 00:55:53
    B>* 10.0.0.253/32 [20/0] via fe80::4638:39ff:fe00:7, swp44, 00:55:50
    C * 10.1.3.0/24 is directly connected, vlan13-v0, 00:55:53
    C>* 10.1.3.0/24 is directly connected, vlan13, 00:55:53
    C * 10.2.4.0/24 is directly connected, vlan24-v0, 00:55:53
    C>* 10.2.4.0/24 is directly connected, vlan24, 00:55:53
    C>* 169.254.1.0/30 is directly connected, peerlink.4094, 00:55:53
    
    
    show ipv6 route
    ===============
    Codes: K - kernel route, C - connected, S - static, R - RIPng,
           O - OSPFv3, I - IS-IS, B - BGP, N - NHRP, T - Table,
           v - VNC, V - VNC-Direct, A - Babel, D - SHARP,
           > - selected route, * - FIB route
    
    C * fe80::/64 is directly connected, vlan24-v0, 00:55:53
    C * fe80::/64 is directly connected, vlan24, 00:55:53
    C * fe80::/64 is directly connected, vlan13-v0, 00:55:53
    C * fe80::/64 is directly connected, vlan13, 00:55:53
    C * fe80::/64 is directly connected, bridge, 00:55:53
    C * fe80::/64 is directly connected, peerlink.4094, 00:55:53
    C * fe80::/64 is directly connected, swp52, 00:55:53
    C * fe80::/64 is directly connected, swp51, 00:55:53
    C>* fe80::/64 is directly connected, swp44, 00:55:53
    

   

Ping from Server01 to Server04 on (different VLANs and racks), and ping from server01 to Internet Router

 

               
                               
    cumulus@server01:~$ ping 10.2.4.104
    PING 10.2.4.104 (10.2.4.104) 56(84) bytes of data.
    64 bytes from 10.2.4.104: icmp_seq=1 ttl=62 time=6.56 ms
    64 bytes from 10.2.4.104: icmp_seq=2 ttl=62 time=2.63 ms
    64 bytes from 10.2.4.104: icmp_seq=3 ttl=62 time=2.17 ms
    ^C
    --- 10.2.4.104 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2002ms
    rtt min/avg/max/mdev = 2.178/3.792/6.563/1.969 ms
    cumulus@server01:~$ ping 171.16.1.1
    PING 172.16.1.1 (172.16.1.1) 56(84) bytes of data.
    64 bytes from 172.16.1.1: icmp_seq=1 ttl=62 time=2.53 ms
    64 bytes from 172.16.1.1: icmp_seq=2 ttl=62 time=2.92 ms
    









    
    



