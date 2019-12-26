## VRF Route leaking

### Topology
<pre>


                                       +-----------------------------------------+   
                                       |                                         |
                                       |                                         | 
                          .2 +-------------+    +--------+   +----------+    +------------+.2
                   +---------|ethernet 01  |----|   vrf  |   |  global  |----|ethernet 00 |----------+ 
                   |         +-------------+    +--------+   +----------+    +------------+          |
             192.168.1.0/24            |                                         |            172.16.1.0/24
                   |                   |                  router                 |                   |
                   |.1                 |                                         |                   | .1  
           +--------------+            +-----------------------------------------+             +-----------+ 
           |   network A  |                                                                    | network B | 
           +--------------+                                                                    +-----------+ 

</pre>

## Cisco ios
### Method 1
Using the `vrf receive` command and policy based routing
<pre>
interface ethernet 01
ip vrf forwarding VRF
ip address 192.168.1.2 255.255.255.0
 
interface ethernet 00
vrf receive VRF
ip address 172.16.1.2 255.255.255.0
ip policy route-map MAP-GLOBAL-to-VRF
 
! permit all the traffic here which needs to go to vrf
ip access-list extended ACL-GLOBAL-to-VRF
 permit ip host <sorce from global> host <destination in vrf>

route-map MAP-GLOBAL-to-VRF permit 10
match ip address ACL-GLOBAL-to-VRF
set ip vrf VRF next-hop 192.168.1.1
 </pre>
 
The command `ip vrf receive VRF1` adds the connected subnet of global interface to the vrf routing table
and traffic from global to vrf is policy routed by the route-map applied under the global interface.


### Method 2
Using static routes between the two tables
<pre>
ip route <network A> <mask> ethernet 01
ip route vrf VRF <network B> <mask> ethernet 00 172.16.1.1 global
</pre>

We can also combine the trick used in method 1 and method 2 `vrf receive` command and static route

<pre>
ip route <network A> <mask> <interface 01>
interface interface 00
  vrf receive VRF
  ip address  172.16.1.2 255.255.255.0
</pre>

### Method 3
This method uses tunnel to connect the vrf to the global table as shown in below schematic

<pre>

                          +-----------------------------------------------------------------------+                         
                          |                                                                       |                         
                          |                                                                       |                         
               .2 +-------------+       +------------+             +-------------+         +------------+  .2               
        +---------|ethernet 01  |-------|   vrf      |             |   global    |---------|ethernet 00 |------------+      
        |         +-------------+       +------------+             +-------------+         +------------+            |      
  192.168.1.0/24          |                                                                       |           172.16.1.0/24 
        |                 |              +----------+   10.10.10.0/24   +-----------+             |                  |      
        |.1               |   +----------|-tunnel 0 --------------------- tunnel 1--|---------+   |                  |      
        |                 |   |          +----------+.1              .2 +-----------+         |   |                  | .1   
        |                 |   |  loopback 0   |                                |   loopback 1 |   |            +-----------+
+--------------+          |   +---------------+                                +--------------+   |            |           |
|              |          |                                                                       |            | network B |
|  network A   |          |                                                                       |            |           |
|              |          |                                                                       |            +-----------+
+--------------+          |                              router                                   |                         
                          +-----------------------------------------------------------------------+   


</pre>

<pre>
interface Loopback2
 description source for tunnel 2 in vrf
 ip address 10.2.2.2 255.255.255.255

interface Loopback1
 description source for tunnel 1 in global
 ip address 10.1.1.1 255.255.255.255

interface Tunnel2
 description internal connection vrf to global
 vrf forwarding VRF
 ip address 10.10.10.2 255.255.255.0
 tunnel source Loopback2
 tunnel destination 10.1.1.1 ! Loopback2

interface Tunnel1
 description internal connection global to vrf
 ip address 10.10.10.1 255.255.255.0
 tunnel source Loopback1
 tunnel destination 10.2.2.2 ! Loopback2
</pre>

We can then use static routes between the two tables
<pre>
ip route <destination in vrf> Tunnel1 name to-vrf-VRF
ip route vrf VRF <destination in global> Tunnel0 name to-global-table
</pre>

We also can do dynamic routing thru the tunnel interface connecting the two tables
<pre>
interface Tunnel1
 ip ospf 1 area 0

interface Tunnel2
 ip ospf 1 area 0

</pre>


## Cisco ios-xr
### Method 1
Leaking routes between vrf and global using bgp
<pre>
vrf VRF
 address-family ipv4 unicast
  import from default-vrf route-policy global-to-vrf
  export to default-vrf route-policy vrf-webserver-to-global
 !
!
route-policy global-to-vrf-webserver
  done
end-policy
!
route-policy vrf-webserver-to-global
  done
end-policy
!
router bgp 65500
 address-family ipv4 unicast
  redistribute connected
 !
 address-family vpnv4 unicast
 !
 vrf VRF
  rd 100:1
  address-family ipv4 unicast
   redistribute connected
  !
 !
</pre>

This will add all bgp routes from the global table to the vrf routing table and vice versa
also because of the redistibute connected statement all the connected routes from one of the two tables will be leaked
to the other


### Method 2
Using static routes

<pre>
router static
 address-family ipv4 unicast
  192.168.1.0/24 vrf webserver 192.168.1.1
</pre>

> To be added: how to add static route in vrr to point to global


### Method 3
Using BGP and route-targets to leak between two vrfs on the same router
<pre>

                                       +-----------------------------------------+   
                                       |                                         |
                                       |                                         | 
                          .2 +-------------+    +--------+   +----------+    +------------+.2
                   +---------|ethernet 01  |----|   vrf  |   |   vrf    |----|ethernet 00 |----------+ 
                   |         +-------------+    +--------+   +----------+    +------------+          |
             192.168.1.0/24            |                                         |            172.16.1.0/24
                   |                   |                  router                 |                   |
                   |.1                 |                                         |                   | .1  
           +--------------+            +-----------------------------------------+             +-----------+ 
           |   network A  |                                                                    | network B | 
           +--------------+                                                                    +-----------+

</pre>

<pre>
vrf RTI
 address-family ipv4 unicast
  import route-target
   100:2
   100:1
  !
  export route-target
   100:2
  !
 !
!
vrf VRF
 address-family ipv4 unicast
  import route-target
   100:1
   100:2
  !
  export route-target
   100:1
  !
 !
!
router bgp 65500
 address-family ipv4 unicast
 !
 address-family vpnv4 unicast
 !
 vrf RTI
  rd 100:2
  address-family ipv4 unicast
   redistribute connected
  !
 !
 vrf VRF
  rd 100:1
  address-family ipv4 unicast
   redistribute connected
  !
 !
!
</pre>


## Juniper junos

<pre>

                                       +-----------------------------------------+   
                                       |                                         |
                                       |                                         | 
                          .2 +-------------+    +--------+   +----------+    +------------+.2
                   +---------|ethernet 01  |----|   vrf  |   |  global  |----|ethernet 00 |----------+ 
                   |         +-------------+    +--------+   +----------+    +------------+          |
             192.168.1.0/24            |                                         |            172.16.1.0/24
                   |                   |                  router                 |                   |
                   |.1                 |                                         |                   | .1  
           +--------------+            +-----------------------------------------+             +-----------+ 
           |   network A  |                                                                    | network B | 
           +--------------+                                                                    +-----------+
           
</pre>

### Method 1
Using rib-groups and static routes from vrf to global
<pre>
pycon@pycon-junos# show routing-instances
VRF {
    routing-options {
        static {
            route 172.16.1.0/24 next-table inet.0;
        }
    }
}
</pre>

Note: This can only be done in one direction between two tables,
in this case we have used next-table in a vrf to point to the global routing table
we cant do the same on the global to point to the same vrf as shown below

<pre>
root# show | compare                                                        
[edit routing-options]
+   static {
+       route 192.168.1.0/24  next-table VRF.inet.0;
+   }
[edit routing-instances VRF routing-options static]
+      route 172.16.1.0/24 next-table inet.0;

[edit]
root# commit 
error: [rib inet.0 routing-options static]
    next-table may loop
error: configuration check-out failed

root# run show version brief 
Model: firefly-perimeter
JUNOS Software Release [12.1X47-D15.4]
</pre>

Therefore to leak routes from the vrf to global we use rib-groups,
The configured rib group in the following config snipet takes routes from the VRF to global table when applied to the respective protocol
you can specify a policy to define what prefixes are leaked from the vrf to global

<pre>
pycon@pycon-junos> show configuration routing-options
static {
rib-groups {
    vrf-to-global-v4 {
        import-rib [ VRF.inet.0 inet.0 ];
    }
}
</pre>

Below config shows different way of applying the rib-groups for different protocols
<pre>
pycon@pycon-junos# show routing-instances
attacker {
    routing-options {
        interface-routes {
            rib-group {
                inet vrf-to-global-v4;
            }
        }
        static {
            rib-group vrf-to-global-v4;
        }
    }
    protocols {
        bgp {
            group bgp {
                family inet {
                    unicast {
                        rib-group vrf-to-global-v4;
                    }
                }
            }
        }
        ospf {
            rib-group vrf-to-global-v4;
        }
        rip {
            rib-group vrf-to-global-v4;
        }
        isis {
            rib-group inet vrf-to-global-v4;
        }
    }
}
</pre>

### Method 3
Using RIB Groups to leak routes between vrf and global.
In the example only interface routes will be leaked between the two table

<pre>
root# show routing-options
interface-routes {
    rib-group inet global-to-vrf;
   }
rib-groups {
    vrf-to-global {
        import-rib [ VRF.inet.0 inet.0 ];
    }
    global-to-vrf {
        import-rib [ inet.0 VRF.inet.0 ];
    }
}
root# show routing-instances VRF
routing-options {
    interface-routes {
        rib-group inet vrf-to-global;
    }
}
</pre>

### Method 3
Using instance import. This method is only applicable to vrfs of type virtual-routers.

<pre>
root# show policy-options 
policy-statement global-to-vrf {
    from instance master;
    then accept;
}
policy-statement vrf-to-global {
    from instance VRF;
    then accept;
}

root# show routing-options 
instance-import vrf-to-global;

root# show routing-instances VRF 
instance-type virtual-router;
routing-options {
    instance-import global-to-vrf;
}
</pre>

### Method 4
Using RIB Groups between two vrfs on the same router

<pre>

                                       +-----------------------------------------+   
                                       |                                         |
                                       |                                         | 
                          .2 +-------------+    +--------+   +----------+    +------------+.2
                   +---------|ethernet 01  |----|   vrf  |   |    vrf   |----|ethernet 00 |----------+ 
                   |         +-------------+    +--------+   +----------+    +------------+          |
             192.168.1.0/24            |                                         |            172.16.1.0/24
                   |                   |                  router                 |                   |
                   |.1                 |                                         |                   | .1  
           +--------------+            +-----------------------------------------+             +-----------+ 
           |   network A  |                                                                    | network B | 
           +--------------+                                                                    +-----------+
           
</pre>

In the example only interface routes will be leaked between the two vrfs
In this config snippet the vrf names are VRF and RTI

<pre>
root# show routing-options rib-groups 
rti-to-vrf {
    import-rib [ RTI.inet.0 VRF.inet.0 ];
}
vrf-to-rti {
    import-rib [ VRF.inet.0 RTI.inet.0 ];
}

root# show routing-instances RTI 
routing-options {
    interface-routes {
        rib-group inet rti-to-vrf;
    }
}
root# show routing-instances VRF
routing-options {
    interface-routes {
        rib-group inet vrf-to-rti;
    }
}

</pre>

### Method 5
Using route-target and auto-export to leak routes between two vrfs on the same router

<pre>
root# show routing-instances 
RTI {
    instance-type vrf;
    vrf-target target:100:1;
    routing-options {
        auto-export;
    }
}
VRF {
    instance-type vrf;
    vrf-target target:100:1;
    routing-options {
        auto-export;
    }
}       
</pre>
