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
This type of VPWS uses BGP instead of LDP, in fact this implementation offers auto discovery options in which the neighbours participating in a layer 2 connection can be discovered automatically and sessions will be established using BGP as a signalling protocol. Hence BGP is used as both neighbor discovery protocol and signalling protocol.
To accommodate these features a new BGP NLRI was introduced to the NLRI is l2vpn. This design uses the concept of site-ids to establish sessions, the site ids are supposed to be on contagious blocks also uses extended BGP communities to carry neighbour and layer 2 route information such as route-target, route-distinguisher, site-id and remote-site-id.

### 3. VPWS BGP Auto discovery and LDP signalling FEC129;
Very much like the previous design, this type of VPWS uses BGP for neighbour discovery only then LDP is used for signalling. This implementation uses BGP NLRI l2vpn for auto-discovery-only.

# Point to multipoint L2VPNs
The point to point L2VPNS are technically referred to as Virtual Private Line Service (VPLS) in this type of configuration the provider networks acts as a switch and the PEs involved will be learning mac-address information.

### 1. VPLS Manual discovery and LDP signalling FEC128 
This implementation requires neighbouring routers participating in the VPLS instance to be explicitly configured on each of the participating routers and it uses LDP as the signalling protocol.

### 2. VPLS BGP Auto discovery and LDP signalling FEC129
This is an enhancement to the above implementation that offers auto discovery of neighbors participating in a VPLS instance. Uses the BGP NLRI l2vpn for auto-discovery-only as well as extended BGP communities such as route-distinguishers and route targets.

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

