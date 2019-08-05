# Route-distinguisher
The Route Distinguisher (rd) is used in MPLS L3-VPNs to identify unique routes corresponding to different VRFs. An IPv4 address is 32 bits, with MPLS L3-VPNs several customers are likely to use the same IP networks (similar subnets). 
Say we have two different customers connecting to the same PE-router. That is customer A who uses 192.168.1.0/24 as LAN subnet and by the grace of God another customer B also uses 192.168.1.0/24 for LAN subnet, we therefore need to explicitly be able to identify each of these routes uniquely as they come from different sources. Since the egress PE-router will be sending these VPN routes updates via mBGP to other routers, the other routers must have a way of separating routes from customer x to those of customer y even if the subnets are similar. 
<p>The route-distinguisher helps in this by adding a rd-value (64-bit) on the IPv4 address (32-bit) thus making a 96-bit VPNv4 prefix, extending the formal network  address for all routes in a particular VRF using a unique value (route-distiguisher)
The rd just assists to differentiate similar routes from different VRFs. It has nothing to do with the VPN in itself. It only provides a work around the BGP's attitude to identical-looking networks and exists purely within the BGP communication
As stated earlier consider the network 192.168.1.0/24 from customer X and from customer Y. These two networks are not the same. In order to make them be treated as different networks by BGP we assign the rt 100:1 to customer x and rt 100:2 to customer y through VRF configuration on the Pe-router. This will make their respective VPN routes be</p>  

> - Customer x: **100:1**:192.168.1.0/24   
> - Customer y: **100:2**:192.168.1.0/24  

Note the routes include the rd value. These routes will be sent as such to other PE-routers
The values of route distinguishers can be made flexible according to the design.
Nevertheless there is a special format for route-distinguisher usually in the form of x:y where x can be the ASN or IP address and y the VPN-identifier.
Technically there are different types of route-distinguishers namely type 0, type 1 and type 2.
Type 0 and 1 route-distinguishers are used for MPLS L3-VPNs (vpnv4)
Route-distinguishers must be unique for different VRFs on the same PE-router, they can be similar or different for the same VRF across different PE-routers.
To learn more about route-distinguisher types visit  
> https://sites.google.com/site/amitsciscozone/home/important-tips/mpls-wiki/route-distinguisher-its-types


# Route-target:
The rt is a BGP attribute of a route, it is an extended community attribute. Being an attribute means that this value is a property of the route that specifies how the route shall be processed but does not form a part of the route's network address as the route-distinguisher does.
The rt in particular is used to say into  which VRFs the particular route gets imported. If a route has community attributes of 1:1 and 1:2 as configured in the VRF, that the route can be imported to any  VRF that imports routes with either 1:1 or 1:2


So the rt says into which VRF a route can be imported.  The rd helps BGP understand that network 192.168.1.0/24 from one VRF is not the same  as the network 192.168.1.0/24 from another VRF. Both RT and RD have the same format (x:y) but that is their only similarity. rd is assigned as a part of the network address within BGP updates while  rt is carried as attributes of the networks. That is why the rd may or may not be the same in two corresponding VRFs on two different PEs - it actually does not matter. It is the rt that defines what routes get into the VRF, which also doesn’t have to match between PEs

### Let us revisit the example for better understanding

Consider two customer A and B, thus two VRFs on a single router VRF-A and VRF-B, each of them containing a single network 192.168.1.0/24.
Without an rd. If BGP sends the routes from this router to a peer, it sends them in  some sequence.
Say the first routes sent are the ones from  VRF-A, then routes from another VRF-B, are advertised secondly.
VRF-A uses rt of 100:1, VRF-B uses rt of 100:2
The BGP peer gets updated sequentially first, the update says there is a network 192.168.1.0/24 with the rt set to 100:1.
This BGP peer then places the route into the  corresponding VRF-A.  
<p>Now, a second update comes in (or a second entry  from the single update is processed), and it says that  there is the same network 192.168.1.0/24  with the rt set to 100:2.  The BGP peer then considers the second  update to be a  replacement of the previous update - the same network  but different attributes. So it removes the network 192.168.1.0/24  from the VRF-A and add it to VRF-B.
This happens because in BGP, the network and its netmask are similar (from both VRFs it is 192.168.1.0/24). An update to a network is performed by sending the update about this network again, with new  attributes explicitly specified. There is no need to first withdraw the route received first (from VRF-A) and because BGP does not see the difference between the network  192.168.1.0/24 from VRF-A and 192.168.1.0/24 from VRF-B, it gets confused because it  thinks that the information about the same network  192.168.1.0/24 from VRF-A (initially received with rt 100:1) simply got updated when the update from VRF-B with rt 100:2 is received.
The rd changes this by prepending the rt value to the VPN routes of each VRF. Thus routes become</p>  

> - Customer x: 100:1:192.168.1.0/24
> - Customer y: 100:2:192.168.1.0/24  

Now BGP will be able to differentiate between 192.168.1.0/24 from VRF-A to that of VRF-B using the rt prefix
In service provider networks normally same rd and rt are used across different PEs. This can be changed and we can have different rd on different PEs, the use of different rd allows for load-sharing and faster convergence.
