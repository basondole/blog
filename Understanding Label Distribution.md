# LDP & MPLS forwarding
## Common terms in MPLS 
- FEC Forwarding Equivalent Class is the equivalent of an IP subnet or IP prefix  
- LSP Label switced path is an end-to-end path composed of LSR from the ingress to the egress LSRs
- LSR Label Switch Router is router that participates in the exchange and distribution of labels and is part of a LSP  

## Operation
An LDP neighbor announces all LDP enabled interfaces (direct connected IP networks) to its LDP neighbors
With Cisco IOS each router (LSR) then assigns labels to all networks that are learnt by the means of IGP (OSPF or ISIS) Cisco IOS always prefers the labelled path if an entry exists in the LFIB which can be accessed via
`sh mpls forwarding-table`

JunOS only assigns label to host routes (/32) learnt via IGP. By default in JunOS LDP has a preference of 9 and routes learnt via LDP are stored in a separate routing table called inet.3 which is used by BGP to resolve next hops for MPLS VPNs and it is not used by normal routing. So the inet.3 table is populated by MPLS protocols (LDP) and used by BGP for next-hop resolution for label path, same as it would use inet.0 for IP path
	`sh route table inet.3`

The default behaviours of both Cisco IOS and JunOS can be changed.

## Label Control Mode:

-	Ordered Control:  
In this approach, an LSR doesn’t advertise a FEC unless it’s the egress LSR for that FEC or until it has received a label for the FEC from its downstream peer. This is used by RSVP, LDP (JunOS)
-	Independent Control:  
This means that the LSR sending the label acts independently of its downstream peer. It does not wait for a label from the downstream LSR before it sends a label to its peers. This mode has the potential of black holing the traffic. For instance, when operating in independent Downstream on Demand mode, an LSR may answer requests for label mappings immediately, without waiting for a label mapping from the next hop. This mode is used by LDP (IOS/IOS-XR)

## Enhancements
### JunOS
LDP IGP sync
Makes an IGP interface that is not running LDP to have a higher metric so as it is not used and break LSPs if it becomes the preferred path
```
set protocols isis interface all point-to-point
set protocols isis interface all ldp-synchronization
show isis interface extensive | match ldp
```

LDP IGP metric tracking
LDP metric is always 1 by default, metric-tracking makes LDP use IGP metric in JunOS  
`set protocol ldp track-igp-metric`

LDP egress-policy  
Make JunOS create and assign label to a non /32 or non IGP route  
`set protocol ldp egress-policy <POLICY>`

MPLS forwarding  
By default only BGP uses the inet.3 table for MPLS. To make normal routing also use mpls  
	`set protocols mpls traffic-engineering mpls-forwarding`

### Cisco
Make router advertise labels for specific prefixes  
`mpls ldp advertise-labels for ACL-NAME`

# How is the best label selected?
First all connected prefixes are assigned implicit-null local label. in Junos are assigned output label 3 or pop-tag then the router assigns local labels to all the routes learned via IGP.  
Note when the local label is implicit-null the prefix is not installed in the LFIB

For outgoing labels the below process is followed:  
Cisco IOS
1. the router checks the IP routing table for the next hop from
	 sh ip route a.b.c.d
2. the router checks the ldp neighbors to see which neighbor has that next hop from below
	 sh mpls ldp nei | i <ip-of-next-hop>
3. the router chooses the label from the ldp neighbor that has the next hop from below
	 sh mpls ldp binding | be a.b.c.d
4. the router pushes the selected label as an outgoing label on the prefix a.b.c.d
	 sh mpls forwarding table | i a.b.c.d
the command above shows the prefix its local/outgoing label egress interface and next-hop address

NB:
> only routes that are advertised in IGP get a remote binding.  
> routes connected to interfaces that are ldp enbled do not get remote binding


JunOS  
> Only prefixes with /32 are assigned labels. The local assigned labels are termed output labels while labels from neighbors are termed input labels.  
1. the router checks the IP routing table for the next hop
	 `sh route table inet.0 a.b.c.d`
2. the router checks the ldp neighbors to see which neighbor has that next hop
	 `sh ldp session details | m <ip-of-next-hop>`
3. the router chooses the label from the ldp neighbor that has the next hop
	 `sh ldp database`
4. the router pushes the selected label as an outgoing label on the prefix a.b.c.d
	 `sh route table inet.3 | find <prefix|incoming label>`
	OR 
`sh route table mpls.0 | find <local-label> `
this command shows labels only without the respective IP prefixes

# Label operations for MPLS VPNs Examples

<img width="790" alt="Labels in MPLS VPN" src="https://user-images.githubusercontent.com/50369643/62413165-59e10b80-b615-11e9-88f1-1da53167d496.png">


PE1 and PE2 are members of VRF_A both send routes to each other via route reflector. Below is an analysis of what happens in details.
<pre>
PE2#sh bgp vpnv4 unicast all labels  
----output omitted------
   Network          Next Hop      In label/Out label     
Route Distinguisher: 100:199 (VRF_A)
   192.168.22.0     10.188.128.36   nolabel/55   
   192.168.199.0    10.41.199.2     126/nolabel 
</pre>

The above output shows the VPN labels. PE1 with IP 10.188.128.36  is the ingress router for prefix  192.168.22.0 from PE1; 55 is outgoing label meaning this PE advertises to other PEs to use this label for traffic destined to VRF_A on PE1
PE2 is the egress router for prefix 192.168.199.0 which it receives from a CE (10.41.199.2 via dynamic routing). PE2 has a local in label 126 meaning this PE will announce this label to other PEs to use it as outgoing label. When PE2 receives packets it will check the local LFIB to see what to do with this label (as shown later this will remove the label and forward to CE) From what is seen so far inlabels are only assigned on locally originated routes; BGP queries the LFIB to get the label to use as inlabel. Note that a VPNv4 route will be distributed only if it has a valid MPLS label. Thus in label in one PE is outlabel on the other PE at the end of the LSP

<pre>
PE2#show mpls forwarding-table | i 126 
126        No Label   192.168.199.0/24[V]   \
                                       2098227488    Gi0/0/3.528  10.41.199.2
</pre>
The above shows the in label to be 126 with an no outgoing label since the prefix is local it is just forwarded to the CE via Gi0/0/3.528

For the prefix 192.168.22.0 which is received from PE2 the next hop decides the LSP for remote received routes

<pre>
PE2#sh mpls forwarding-table | i 10.188.128.36  
21         299792     10.188.128.36/32 0             Gi0/0/0.6  172.16.0.245
</pre>

The PE have already exchanged VPNV4 info and the know the respective labels for the routes to be put in VRF using inner labels assigned and exchanged by BGP in this case in label is 126 for the VRF_A on PE2 perspective and the in label is 55 for VRF_A on PE1 perspective
The LSP is just used to forward the packets in MPLS using transport labels assigned by MPLS whc=ich will be stacked on top of VPN labels assigned by BGP (126 and 55 in this example)

Thus traffic from CE (connected to PE2) to (CE connected to PE1) will have 299792 as the transport label which will be swapped along the LSP and will have inner VPN label of 55
At the last LSR before PE1 the transport label will be popped from the label stack (since PE1 advertises an implicit null for 10.188.128.36 as this is directly connected and PE1 is the egress router) 
The packet will then be forwarded to PE1 without the outer transport label but with only the inner VPN label of 55 
PE1 sees this label and it knows it assigned that label to the 192.168.22.0/24 route in a vrf (VRF_A) and forwards to that VRF which then removes the label stack altogether and forwards the packet to the VRF instance where an IP lookup is performed and the packet is forwarded to the CE
<pre>
PE1> show route table mpls.0 | find 55
55                 *[VPN/0] 69w5d 21:57:44
                      to table VRF_A.inet.0, Pop   
</pre>
On PE1 when a packets is sent towards 192.168.199.0 which is a from PE2 two labels are placed the inner VPN label received from PE2 (55) and the outer transport label assigned by MPLS 
<pre>
PE1> show route table bgp.l3vpn.0 rd-prefix 100:199:192.168.199.0/24  active-path              
100:199:192.168.199.0/24                
                   *[BGP/170] 14w4d 02:11:01, MED 100, localpref 100, from 10.188.128.29
                      AS path: ?
                    > to 172.16.1.57 via xe-0/0/0.0, Push 126, Push 300976(top)

</pre>

To see what labels have been assigned by the router to the FEC in VRF and what labels are received from remote PEs for those FEC
<pre>
PE2#sh ip bgp vpnv4 vrf VRF_A labels 
   Network          Next Hop      In label/Out label
Route Distinguisher: 1004:199 (VRF_A)
   192.168.199.0    10.41.199.2     126/nolabel-----------------------advertised routes
   10.41.199.0/30   0.0.0.0         356/nolabel(VRF_A)
   10.56.90.0/29    10.188.128.56   nolabel/27------------------------received routes
   10.79.1.18/32    10.188.128.36   nolabel/55
   10.1.10.0/30     10.188.128.38   nolabel/769315
                    10.188.128.38   nolabel/769267
                    10.188.128.27   nolabel/19
   10.1.10.2/32     10.188.128.38   nolabel/769315
                    10.188.128.38   nolabel/769267
                    10.188.128.27   nolabel/19
</pre>

Juniper assigns one local label per VRF if the VRF is configured with vrf-table-label, routing-instances configured with vrf-table-lable will be assigned one label since they are using a single LSI. When you include the vrf-table-label statement in the configuration of a VRF routing instance, a label-switched interface (LSI) logical interface label is created and mapped to the VRF routing table the current JunOS label allocation implementation for VRFs starts assigning labels from 16 onwards
<pre>
PE2> show route table mpls.0 protocol vpn 
16                 *[VPN/0] 69w3d 23:50:23
                      to table VRF.inet.0, Pop
</pre>

Without the LSI one label is assigned per egress interface thus different interfaces on the same PE and the same VRF will have different labels. Example below interface xe-0/0/3.642 and xe-0/0/3.643 are in same VRF but local labels are assigned per interface.
<pre>
PE1> show route table mpls.0 protocol vpn | find 49856     
498560             *[VPN/170] 13w1d 00:35:35
                    > to 10.36.217.22 via xe-0/0/3.642, Pop      
498576             *[VPN/170] 13w1d 00:35:35
                    > to 10.36.217.18 via xe-0/0/3.643, Pop
</pre>

To see VPN label received from remote PEs for route-distinguisher 199:100
<pre>
junOS> sh route table bgp.l3vpn | ma 199:100
ciscoIOS# sh bgp vpnv4 unicast rd 199:100 labels
</pre>

To see the outgoing transport label or egress interface
<pre>
ciscoIOS# sh mpls for | i <VPN-LABEL>
</pre>
> Note: the VPN in-label and the local-transport-label are similar

MPLS FEC for L3 VRF routes are assigned local VPN label which can be seen as follows
The route in a VRF is 192.168.199.0/24
<pre>
PE2#sh bgp vpnv4 uni all labels |  i 192.168.199.0
   192.168.199.0    10.41.199.2     126/nolabel
PE2#sh mpls forwarding-table labels 126
Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
Label      Label      or Tunnel Id     Switched      interface              
126        No Label   192.168.199.0/24[V]   \
                                       2172694358    Gi0/0/3.528  10.41.199.2 
</pre>

For routes received from remote PEs here FEC 192.168.22.0/24 is received from 10.188.128.36
<pre>
PE2#sh ip bgp vpnv4 vrf VRF_A labels | i 192.168.22.0|Net
   Network          Next Hop      In label/Out label
   192.168.22.0     10.188.128.36   nolabel/55
</pre>
No local label will be assigned but the FEC will have an /out label from remote PE which is 55 in this case
Thus 55 is the L3-VPN-label and the transport label will be that of the next-hop which is 10.188.128.36. This can also be seen in the label stack below

<pre>
PE2#sh mpls for vrf VRF_A 192.168.22.0 detail | i Label
Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
Label      Label      or Tunnel Id     Switched      interface              
        MAC/Encaps=18/26, MRU=9212, Label Stack{299792 55}
</pre>
Here 299792 is the transport label and 55 is the VPN label.
The transport label 299792 should translate to IP 10.188.128.36. This can be confirmed below by checking the ldp binding OR checking the LFIB
<pre>
PE2#sh mpls ldp binding | i lib.*10.188.128.36|10.188.128.35.*299792
  lib entry: 10.188.128.36/32, rev 766
        remote binding: lsr: 10.188.128.35:0, label: 299792

PE2#sh mpls for 10.188.128.36
Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
Label      Label      or Tunnel Id     Switched      interface              
21         299792     10.188.128.36/32 0             Gi0/0/0.6  172.16.0.245
</pre>

# L2MPLS

For L2-MPLS, no outgoing labels are assigned as well
<pre>
PE2#sh l2v atom vc vcid 3019 detail | i Dest|Next|Out|label
    Last label FSM state change time: 17:43:07
  Destination address: 10.188.128.114 VC ID: 3019
    Output interface: Gi0/0/1.64, imposed label stack {589136 299776}
    Next hop: 172.16.0.249
  SSO Descriptor: 10.188.128.114/3019, local label: 282
</pre>


To check for the local label in the LFIB
<pre>
PE2#sh mpls forwarding-table labels 282
Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
Label      Label      or Tunnel Id     Switched      interface              
282        No Label   l2ckt(32)        22392021597   Gi0/0/2.3019  point2point 
</pre>


No outgoing labels are assigned because the LSP is determined by the Next-hop. On the imposed stack {589136 299776} 589136 is the transport-label and 299776 is the L2-VPN label on the other end and according to the imposed stack the transport label of 589136 is assigned by the upstream router for the Next-hop FEC this can be seen in the LFIB
<pre>
PE2#sh mpls forwarding-table | be 589136
57         589136     10.188.128.114/32   \
                                       0             Gi0/0/1.64  172.16.0.249
</pre>

Moreover local VPN (L2 or L3) labels will not be seen in the LIB as shown below, they are only stored in the LFIB
<pre>
PE2#sh mpls ldp binding | i 282
PE2#sh mpls ldp binding | i 126
</pre>


To see the label assigned to L2-VPN
<pre>
PE3> show route table mpls.0 protocol l2circuit 
mpls.0: 118 destinations, 118 routes (118 active, 0 holddown, 0 hidden)
299776             *[L2CKT/7] 17:51:47
                    > via ge-0/0/1.3019, Pop      
ge-0/0/1.3019      *[L2CKT/7] 17:51:47, metric2 1
                    > to 172.18.129.169 via ge-0/0/0.32, Push 282, Push 573872(top)
</pre>


Packets coming from remote PE2 to PE3 with VPN label 299776 will have the label popped and be forwarded to interface  ge-0/0/1.3019 to client
Incoming packets coming from client received on ge-0/0/1.3019 will have the remote VPN label 282 assigned (received from PE2) and the transport label to reach the next-hop is 573872

In contrary to Cisco IOS, these VPN label will be present in the LIB. We can see both input (remote) and  outgoing (local) L2-VPN labels on the LIB
<pre>
PE3> show ldp database | match "(3019|10.188.128.46:0)" 
Input label database, 10.188.128.114:0--10.188.128.46:0
    282     L2CKT NoCtrlWord VLAN VC 3019
Output label database, 10.188.128.114:0--10.188.128.46:0
 299776     L2CKT NoCtrlWord VLAN VC 3019
</pre>


We can check the binding table to see what FEC has that transport label of 573872 as and it should be 10.188.128.46
<pre>
PE3> show ldp database | match "(573872|in|out)"
Input label database, 10.188.128.114:0--10.188.128.29:0
 573872     10.188.128.46/32
</pre>


For L3-VPN in JunOS the VPN-label is only present in the LFIB not in the FIB
<pre>
PE4> show route table mpls.0 protocol vpn  | can match the VRF-name(in case there is lsi) or an interface in the VRF
mpls.0: 108 destinations, 108 routes (108 active, 0 holddown, 0 hidden)
16                 *[VPN/0] 69w5d 18:16:56
                      to table VRF.inet.0, Pop
483168             *[VPN/170] 12w3d 05:32:52
                    > to 10.120.106.2 via ge-0/0/3.701, Pop
</pre>


Similar to Cisco the local L3-VPN labels do not exist in the FIB, if it does it will be associated with a different prefix and will have nothing to do with the MPLS-VPN
<pre>
PE4> show ldp database | match 16 
PE4> show ldp database | match 483168 
</pre>


Example of a FEC in the BTT VRF locally on PE4 is 10.5.24.0/24 and from another PE5 in the same VRF FEC is 10.5.36.0/24
VRF in PE4 will receive the FEC 10.5.36.0/24 with VPN label 19 from PE5 and a transport label will be that of 10.188.128.31 which is the protocol-next-hop (PE5 loopback IP) See below VPN label assigned to all FECs from VRF BTT at PE5
<pre>
PE5> show route table mpls.0 protocol vpn         
19                 *[VPN/0] 71w2d 00:02:41
                      to table BTT.inet.0, Pop       
</pre>


Thus PE4 has to add the VPN label 19 for FECs from PE5 as seen with the push operation below. The transport label is 518912, we can check the label database to see what FEC has that label as input label and it should be 10.188.128.31 (PE5 loopback IP)
<pre>
PE4> show route 10.5.36.0/24 table BTT active | find 10.5.36.0/24 
10.5.36.0/24       *[BGP/170] 2d 00:08:12, MED 0, localpref 100, from 10.188.128.29
                      AS path: I
                    > to 177.16.1.5 via ge-0/0/0.3203, Push 19, Push 518912(top)
                  
PE4> show ldp database | match "(518912|in)"
Input label database, 10.188.128.120:0--10.188.128.47:0
 518912     10.188.128.31/32
</pre>


PE5 will receive FEC 10.5.24.0/24 from PE4 (loopback 10.188.128.120) with a VPN-label 16
<pre>
PE5> show route 10.5.24.0/24 table BTT  active
10.5.24.0/24       *[BGP/170] 1w1d 22:02:53, MED 0, localpref 100, from 10.188.128.29
                      AS path: 64730 I, validation-state: unverified
                    > to 172.16.0.85 via ge-0/0/0.3, Push 16, Push 564176(top)
</pre>


The transport label will be 564176 which should translate to protocol-next-hop 10.188.128.120 (PE4 loopback)
<pre>
PE5> show ldp database | match "(in|564176)"
Input label database, 10.188.128.31:0--10.188.128.29:0
 564176      10.188.128.120/32
</pre>
