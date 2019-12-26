# BGP-IGP VS MPLS-FORWARDING


## inet.0 table
This is the primary unicast routing table used by IPv4. It’s where IGPs will resolve next hops.

## inet.3 table
This is the routing table populated by MPLS protocols such as RSVP or LDP. A lookup here will result in a label being pushed.
`inet.3` is used by BGP to resolve BGP next-hops. BGP examines both `inet.3` and `inet.0`
choosing a next-hop based on the lowest preference value. In case of a tie, `inet.3` is used.

## MPLS traffic-engineering options
You can enable only one of the traffic-engineering statement options at a time
- bgp (default)
- bgp-igp
- bgp-igp-both-ribs
- mpls-forwarding

The default BGP option (traffic-engineering bgp) allows only BGP to use LSPs in its route calculations

### bgp-igp
The `bgp-igp` option causes all `inet.3` routes to be moved to the `inet.0` routing table.
If you configure the `traffic-engineering bgp-igp` statement or the `traffic-engineering bgp-igp-both-ribs` statement,
high-priority RSVP and LDP routes can supersede IGP routes in the `inet.0` routing table.
The effect can be noted when we are matching against a protocol in a routing policy.
IGP routes might no longer be redistributed since they are no longer the active routes.  
LSPs will still be used to resolve BGP next-hops because, BGP can use both tables, it just prefers `inet.3`
With `bgp-igp` all your other traffic gets to use the LSP as well not just BGP traffic  
> Only BGP traffic gets to use the LSP by default so as to support MPLS VPNS, moving everything out of `inet.3` breaks your ability to run MPLS VPNs.

<pre>
root@R6-Junos1> show route 102.102.102.102

inet.0: 21 destinations, 35 routes (21 active, 0 holddown, 0 hidden)
@ = Routing Use Only, # = Forwarding Use Only
+ = Active Route, - = Last Active, * = Both

102.102.102.102/32 *[LDP/9] 00:00:33, metric 1
                    > to 192.168.46.4 via em0.0, Push 28
                    [OSPF/150] 00:00:33, metric 0, tag 0
                    > to 192.168.46.4 via em0.0
</pre>

### bgp-igp-both-ribs
`traffic-engineering bgp-igp-both-ribs` statement does the same as `bgp-igp`
but instead of moving the contents of `inet.3` to `inet.0` it copies it over making the labeled path exist in both
`inet.0` and `inet.3` tables.
As we have seen with the `bgp-igp` option, this means high-priority RSVP and LDP routes can supersede IGP routes in the `inet.0` routing table.
The effect can be noted when we are matching against a protocol in a routing policy.
IGP routes might no longer be redistributed since they are no longer the active routes.  
<pre>
root@Router1> show route 2.2.2.2

inet.0: 15 destinations, 20 routes (15 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

2.2.2.2/32         *[LDP/9] 00:00:18, metric 1
                    > to 10.10.12.2 via em0.0
                    [IS-IS/18] 00:03:01, metric 10
                    > to 10.10.12.2 via em0.0

inet.3: 4 destinations, 5 routes (4 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

2.2.2.2/32         *[LDP/9] 00:00:18, metric 1
                    > to 10.10.12.2 via em0.0
</pre>

### mpls-forwarding

With `mpls-forwarding` RSVP and LDP routes are used for forwarding but are excluded from route selection.
These routes are added to both the `inet.0` and `inet.3` routing tables.
RSVP and LDP routes in the `inet.0` routing table are given a low preference when the active route is selected.
However, RSVP and LDP routes in the inet.3 routing table are given a normal preference
and are therefore used for selecting forwarding next hops.

<pre>
root@R6-Junos1> show route 102.102.102.102

inet.0: 21 destinations, 35 routes (21 active, 0 holddown, 0 hidden)
@ = Routing Use Only, # = Forwarding Use Only
+ = Active Route, - = Last Active, * = Both

102.102.102.102/32 @[OSPF/150] 00:03:56, metric 0, tag 0
                    > to 192.168.46.4 via em0.0
                   #[LDP/9] 00:00:02, metric 1
                    > to 192.168.46.4 via em0.0, Push 28

inet.3: 14 destinations, 14 routes (14 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

102.102.102.102/32 *[LDP/9] 00:00:02, metric 1
                    > to 192.168.46.4 via em0.0, Push 28
</pre>
The outcome for traffic forwarding will be the identical to `bgp-igp` or `bgp-igp-both-ribs` but not for routing operations


BGP is using table `inet.3` to resolve next hops, where as normal IGP routing uses `inet.0`  
Important note is with BGP and `inet.3` if `inet.0` contains a better route (e.g. better preference) 
then BGP would use the `inet.0` route and traffic would not be forwarded on the LSP causing MPLS VPNs to break.



#### Reference
https://www.networkfuntimes.com/moving-lsps-between-inet-3-and-inet-0-on-a-juniper-router/  
http://matt.dinham.net/mapping-traffic-lsp-junos-part-1/  
https://packetcorner.wordpress.com/2012/10/06/difference-between-traffic-engineering-options-bgp-igp-vs-mpls-forwarding/

