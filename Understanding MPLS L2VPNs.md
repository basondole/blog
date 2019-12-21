# MPLS L2VPNs

L2VPNs offer a means to tunnel layer 2 traffic over MPLS network. The L2VPNs can be point to point or point to multipoint. Let’s take a look on what options are available to us

# Point to point L2VPNs
The point to point L2VPNS are technically referred to as Virtual Private Wire Service (VPWS) and they are of three types
-	Martini L2VPN
-	VPWS Kompella using BGP auto-discovery and signalling
-	VPWS BGP Auto discovery and LDP signalling FEC129;

### 1. Martini L2VPNs
This type of VPWS uses LDP as its signalling protocol. They are mostly referred to as layer 2 circuit or cross-connect. This implementation does not offer auto discovery features and both ends of the connections have to be configured manually to establish targeted LDP sessions. There is another implementation of this that uses IP instead of MPLS to tunnel layer 2 traffic called L2TPv3 in Cisco.

### 2. VPWS Kompella using BGP auto-discovery and signalling
This type of VPWS uses BGP instead of LDP, the neighbours participating in a layer 2 connection can be discovered automatically and sessions will be established using BGP as a signalling protocol. BGP is used as both neighbor discovery protocol and signalling protocol.
To accommodate these features a new BGP NLRI was introduced to the NLRI is l2vpn. This design uses the concept of site-ids to establish sessions, the site ids are supposed to be on contagious blocks also uses extended BGP communities to carry neighbour and layer 2 route information such as route-target, route-distinguisher, site-id and remote-site-id.
> Note: Cisco does not support this implementation

### 3. VPWS BGP Auto discovery and LDP signalling FEC129;
Very much like the previous design, this type of VPWS uses BGP for neighbour discovery only then LDP is used for signalling. This implementation uses BGP NLRI l2vpn for auto-discovery-only as well as extended BGP communities; route targets and l2vpn-id.
The presence of the l2vpn-id designates that FEC 129 LDP signaling is used for the routing instance. The absence of l2vpn-id indicates that BGP signaling is used instead

# Point to multipoint L2VPNs
The point to point L2VPNS are technically referred to as Virtual Private Line Service (VPLS) in this type of configuration the provider networks acts as a switch and the PEs involved will be learning mac-address information.

### 1. VPLS Manual discovery and LDP signalling FEC128 
This implementation requires neighbouring routers participating in the VPLS instance to be explicitly configured on each of the participating routers and it uses LDP as the signalling protocol.

### 2. VPLS BGP Auto discovery and LDP signalling FEC129
This is an enhancement to the above implementation that offers auto discovery of neighbors participating in a VPLS instance. Uses the BGP NLRI l2vpn for auto-discovery-only as well as extended BGP communities; route targets and l2vpn-id.
FEC 129 uses BGP autodiscovery to convey endpoint information, so you do not need to manually configure pseudowires. BGP is responsible for distributing the local autodiscovery routes created on each PE device to all other PE devices. LDP is responsible for using the autodiscovery information provided by BGP to set up targeted LDP sessions over which to signal the pseudowires.

### 3. VPLS BGP Auto discovery and BGP signalling
This implementation uses BGP for both neighbors discovery and signalling.

### 4. VPLS Mixed LDP and BGP
This is a hybrid implementation where both VPWS and VPLS are combined to offer the best of both worlds.

# Configuration samples
## Martini L2VPN
Below configuration between Cisco IOS XE and JunOS with existing IP and MPLS underlying configuration  
<img width="646" alt="Martini L2VPN" src="https://user-images.githubusercontent.com/50369643/62411474-f8607300-b5fb-11e9-98cf-f2f2c0c9f447.png">

### Cisco config and verification
<pre>
interface Gi0/0/0.1001
Encapsulation dot1q 1001
xconnect 10.2.2.2 1001 encapsulation mpls

show xconnect interface Gi0/0/0.1001
show mpls l2transport vc 1001
show l2vpn atom vc vcid 1001
</pre>

### JunOS config and verification
<pre>
set interface ge-0/0/0.1001 vlan-id 1001
set interface ge-0/0/0.1001 encapsulation vlan-ccc
set protocols l2cicuit neighbour 10.1.1.1 interface ge-0/0/0.1001 virtual-circuit-id 1001

show l2circuit connection interface ge-0/0/0.1001
</pre>

## Local layer 2 circuit
For L2 circuit whose ends terminate on the same router, that is, two different logical interfaces(or physical) for both ends are configured on the same router there is a number of options to go about it as listed below:
- Interface-switch
- Bridge-domain
- local-switching

### Interface-switch
This JunOS feature allows for configuration of Layer 2 switching cross-connects. The cross-connect is  bidirectional, so packets received on the first interface are transmitted out the second interface, and those received on the second interface are  transmitted out the first.  For Layer 2 switching cross-connects to work, you must also configure MPLS.
The interface-switch can only be configured between two interfaces not more as it does not involve learning of mac addresses and it simply mirrors traffic between the two interfaces if you will.

#### Configuration:
<pre>
 set protocols connections interface-switch SAMPLE interface ge-1/0/8.259
 set protocols connections interface-switch SAMPLE interface ge-1/1/2.259
 set interfaces ge-1/0/8 unit 259 description SITE1
 set interfaces ge-1/0/8 unit 259 encapsulation vlan-ccc
 set interfaces ge-1/0/8 unit 259 family ccc
 set interfaces ge-1/1/2 unit 259 description SITE2
 set interfaces ge-1/1/2 unit 259 encapsulation vlan-ccc
 set interfaces ge-1/1/2 unit 259 family ccc

show connections interface-switch SAMPLE    
  Connection/Circuit                Type        St      Time last up     # Up trans
 SAMPLE                        if-sw       Up      Sep 11 18:05:38           1
   ge-1/0/8.259                     intf  Up
   ge-1/1/2.259                     intf  Up
</pre>


### Local-switching

#### JunOS Configuration
<pre>
set protocols l2circuit local-switching interface ge-0/1/3.4094 end-interface interface ge-0/0/2.4094
set interfaces ge-0/1/3.4094 vlan-id 4094 encapsulation vlan-ccc family ccc
set interfaces ge-0/0/2.4094 vlan-id 4094 encapsulation vlan-ccc family ccc
</pre>

#### Cisco IOS configuration
<pre>
interface GigabitEthernet1.4094
	encapsulation dot1q 4094
interface Gigabitthernet2.4094
 	encapsulation dot1q 4094
connect TEST Gig1.4094 Gig2.4094

sh connection name TEST
Connection: 4 - TEST
 Current State: ADMIN UP
 Segment 1: GigabitEthernet1.4094 up
 Segment 2: GigabitEthernet2.4094 up
</pre>


### VPWS Kompella using BGP auto-discovery and signalling
> Not supported by Cisco
#### JunOS Configuration
<pre>
set protocols bgp group ibgp family l2vpn signaling

set routing-instances l2vpn instance-type l2vpn
set routing-instances l2vpn interface ge-0/0/1.0
set routing-instances l2vpn route-distinguisher 64512:1000
set routing-instances l2vpn vrf-target target:64512:1000
set routing-instances l2vpn protocols l2vpn encapsulation-type ethernet
set routing-instances l2vpn protocols l2vpn site 1 site-identifier 1
set routing-instances l2vpn protocols l2vpn site 1 interface ge0/0/1.0 remote-site-id 2

show route table L2VPN.l2vpn.0
show l2vpn connections up

</pre>


## VPLS Manual discovery and LDP signalling FEC128  
<img width="646" alt="VPLS Manual ldp" src="https://user-images.githubusercontent.com/50369643/62411862-758ee680-b602-11e9-9024-18ec973a257e.png">  

### Cisco IOS config and verification
<pre>
l2 vfi VPLS manual 
 vpn id 1000
 bridge-domain 1000
 mtu 1500
 neighbor 10.2.2.2 encapsulation mpls 
 neighbor 10.3.3.3 encapsulation mpls
 
interface GigabitEthernet1
 no ip address
service instance 1000 ethernet
  encapsulation dot1q 1000
  rewrite ingress tag pop 1 symmetric
  bridge-domain 1000

show l2vpn vfi name VPLS
show mpls l2transport vc 1000
show xconnect name VPLS
show bridge-domain 1000 mac dynamic address
</pre>

### JunOS config and verification
<pre>
set interfaces ge-0/0/2 vlan-tagging
set interfaces ge-0/0/2 encapsulation flexible-ethernet-services
set interfaces ge-0/0/2 unit 1000 encapsulation vlan-vpls
set interfaces ge-0/0/2 unit 1000 vlan-id 1000
set interfaces ge-0/0/2 unit 1000 input-vlan-map pop
set interfaces ge-0/0/2 unit 1000 output-vlan-map push
set interfaces ge-0/0/2 unit 1000 family vpls
set routing-instances VPLS instance-type vpls
set routing-instances VPLS interface ge-0/0/2.1000
set routing-instances VPLS protocols vpls encapsulation-type ethernet
set routing-instances VPLS protocols vpls no-control-word
set routing-instances VPLS protocols vpls no-tunnel-services
set routing-instances VPLS protocols vpls vpls-id 1000
set routing-instances VPLS protocols vpls mtu 1500
set routing-instances VPLS protocols vpls neighbor 10.1.1.1
set routing-instances VPLS protocols vpls neighbor 10.3.3.3

show vpls connections instance VPLS
show vpls mac-table instance VPLS
</pre>



#### References
https://blog.marquis.co/layer-2-vpns-on-junos/  
https://mellowd.co.uk/ccie/l2vpn-on-junos-using-cccmartinikompella/  
https://www.inetzero.com/vpls-some-simples-configurations/
https://networkzblogger.com/2017/05/03/mpls-l2vpn-configuration-example-juniper-l2circuit-mpls-l2vpn-tutorial-what-is-l2vpn-l2vpn-wiki-how-l2vpn-works-l2vpn-juniper-kompella/

