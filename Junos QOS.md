# Implementing QoS in junos
Quality of Service (QoS) is the manipulation of traffic such that a network device, such as a router or switch, forwards it in a fashion consistent with the required behaviors of the applications generating that traffic. In other words, QoS enables a network device to differentiate traffic and then apply different behaviors to the traffic.
QoS includes these capabilities:  
- Prioritizing traffic over other traffic based on protocol, address and port number.  
- Filtering traffic upon ingress or egress.  
- Controlling the allowed bandwidth transmitted or received on the device.  
- Reading and writing QoS behavior requirements in the packet header.  
- Controlling congestion so that the device sends the highest priority traffic based on scheduler priorities.  
- Controlling packet loss using algorithms, so that the device knows the packets to drop or process.


Below diagram shows the building blocks of QoS
 

## Classification
Classifiers map traffic to a forwarding class at ingress. Classifiers match ingress traffic that already has COS markings and depending on the observed mark traffic is assigned to a forwarding class (queue) this type of classification is called Behavior Aggregate (BA)
Classifiers can also match on protocol, port, addresses etc this classification type is called Multifield classification (MF) BA is considered to be good for high volume devices  and more efficient than MF.
Fixed Classification is another form of classification. This method assigns a single forwarding class to all ingress packets with no regard to their markings (if any)   
CoS marking can be of different types as described below  
-	IPv4 packet: Uses DSCP(6 bit) with up to 64 classes of service
-	IPv6 packet: Uses DSCP(6 bit) with up to 64 classes of service
-	Ethernet frame: Uses priority code point (3 bit) with up to 8 classes of service
-	MPLS:  Uses EXP bits (3 bit) with up to 8 classes of service

These marking are used with BA classification to assign packets to desired forwarding classes. Classifiers also define the Packet Loss Probability (PLP) which determines the likelihood of packet being dropped under congestion essentially helping policers and schedulers determine which traffic to drop first during congestion.
In below example ingress packets on interface    ge-0/0/0  witch DSCP marking 011010 will be assigned to network-control class and assigned PLP of high while ingress labeled packets with EXP marking 001 will be assigned to expedited-forwarding class and assigned a PLP of high.
Different classifiers can be assigned on an interface to match different type of markings schemes such as 802.1/DSCP/EXP etc

Configuring and applying a BA classifier on an interface
<pre>
paul@big# show class-of-service                             
classifiers {
    dscp CLASSIFIER-CUSTOM-DSCP {
        import default;
        forwarding-class network-control {
            loss-priority high code-points 011010;
        }
    }
}
interfaces {
    ge-0/0/0 {
        unit 0 {
            classifiers {
                dscp CLASSIFIER-CUSTOM-DSCP;
            }
        }
    }
}
</pre>

The keyword import default causes the classifier to inherit unspecified values from the default class. For example in this case we have only specified code point 011010 to be mapped to network-control class and since DSCP has 64 values the remaining 63 values (which we have not specified) will be mapped to different classes following the default DSCP scheme

Verification
<pre>
paul@big# run show class-of-service interface ge-0/0/0.0    
  Logical interface: ge-0/0/0.0, Index: 70
    Object                  Name                   Type                    Index
    Classifier              CLASSIFIER-CUSTOM-DSCP dscp                    62414
</pre>

BA classifier without import statement
<pre>
paul@big# run show class-of-service classifier name CLASSIFIER-CUSTOM-DSCP      
Classifier: CLASSIFIER-CUSTOM-DSCP, Code point type: dscp, Index: 62414
  Code point         Forwarding class                    Loss priority
  011010             network-control                     high
</pre>

BA classifier with import statement
<pre>
paul@big# run show class-of-service classifier name CLASSIFIER-CUSTOM-DSCP    
Classifier: CLASSIFIER-CUSTOM-DSCP, Code point type: dscp, Index: 62414
  Code point         Forwarding class                    Loss priority
  000000             best-effort                         low         
  000001             best-effort                         low         
  001011             best-effort                         low         
  001100             assured-forwarding                  high        
{Output omitted}      
  001110             assured-forwarding                  high        
  010010             best-effort                         low         
  010011             best-effort                         low         
  010100             best-effort                         low
  011010             network-control                     high
{Output omitted}      
</pre>

Configuring and applying a MF
<pre>
paul@big# show firewall 
filter APPLY-COS-MARK {
    term 1 {
        from {
            source-address {
                192.168.56.0/24;
            }
        }
        then {
            forwarding-class assured-forwarding;
            accept;
        }
    }
    term all-other-traffic {
        then accept;
    }
}
paul@big# show interfaces ge-0/0/0.0  
family inet {
    filter {
        input APPLY-COS-MARK;
    }
}
</pre>

Verification
<pre>
paul@big# run show class-of-service interface ge-0/0/0.0  
  Logical interface: ge-0/0/0.0, Index: 70
    Object                  Name                   Type                    Index
    Classifier              ipprec-compatibility   ip                         13 


Configuring fixed classification
paul@big# show class-of-service                                
interfaces {
    ge-0/0/1 {
        unit 1000 {
            forwarding-class expedited-forwarding;
        }
    }
}
</pre>

Verification
<pre>
paul@big# run show class-of-service interface ge-0/0/1.1000    
  Logical interface: ge-0/0/1.1000, Index: 76
    Object                  Name                   Type                    Index
    Classifier              expedited-forwarding   fixed                       1
</pre>


## Queues
Forwarding classes determine the queue to which a packet is assigned. By default there are 4 forwarding classes in junos  
1.	BE (Best effort) Basic services 
2.	EF (Expedited Forwarding) Premium services
3.	AF (Assured forwarding)
4.	NC (Network control) Control Traffic

<pre>
paul@big# run show class-of-service forwarding-class
Forwarding class        ID      Queue  Policing priority    SPU priority
  best-effort            0        0           normal           low    
  expedited-forwarding   1        1           normal           low    
  assured-forwarding     2        2           normal           low    
  network-control        3        3           normal           low    
</pre>

Configuring forwarding classes
<pre>
paul@big# show class-of-service forwarding-classes
class MYOWNCLASS queue-num 4;

paul@big# run show class-of-service forwarding-class
Forwarding class        ID      Queue  Policing priority    SPU priority
  best-effort            0        0           normal           low    
  expedited-forwarding   1        1           normal           low    
  assured-forwarding     2        2           normal           low    
  network-control        3        3           normal           low    
  MYOWNCLASS             4        4           normal           low
</pre>

After creating a forwarding class we use classifiers to assign ingress packets to that forwarding class

## Scheduling
Schedulers define the prioritization properties of forwarding classes (queues) on egress. These properties include  
   - Order in which packets are transmitted
   - Rate at which packets are transmitted
   - Number of packets that can be transmitted
   - Differential treatment of packets in the event of congestion

### Components of scheduling
#### Transmission rate
This component specifies the amount of bandwidth allocated to a queue. Transmission rate has the following control options  
1.	Exact, where traffic exceeding transmission rate is buffered (functions as queue's shaper)
2.	Rate-limit, where traffic exceeding transmission rate is dropped (functions as hard policer)
If none of the control options is specified queue can exceed transmission rate if unused bandwidth exists. The scheduler shares unallocated bandwidth equally among eligible queues in a round-robin fashion on per packet basis

#### Priority
Priority defines relative importance of a queue compared to other queues by ensuring certain queues are served before other queues when congestion occurs. There a five priority levels  
1.	low
2.	medium-low
3.	medium-high
4.	high
5.	strict-high  
Strict-high level has the top priority; traffic in this level is considered to always be in-profile and receives precedence over all other queues except queue with high priority. Other queues are serviced using weighted round-robin
The strict high queue may starve lower priority queues, for this reason we configure transmission-rate with rate-limit to stop strict-high queues from using the entire interface bandwidth and starve other queues. In my opinion it is very important you evaluate what traffic you want to assign to this queue and strictly specify the bandwidth allocated to the strict high queue.

### Delay buffers
The egress interface has a buffer with finite capacity, when there is congestion packets are temporarily stored in the buffer (memory) before getting transmitted. This component specifies the amount of data that can be stored when congestion occurs and to be transmitted when bandwidth utilization goes down, large buffer means many packets stored. The buffer size is determined by the interface hardware.
     Example: 1Gbps port with 100ms buffer size provides a 12.5Mbytes of stored bytes
              1Gbps x 100ms = 100Mb or 12.5MBytes

Delay buffer can be configure as  
-	Percentage of ports available buffer. This option allows buffer to expand when queue exceeds transmission rate  
    `buffer-size (MB) = buffer-size (%) x max-port-buffer-size (ms) x interface-bandwidth (Mbps) / 8`

-	Temporal time value relative to queue's transmission rate (not interface bandwidth). This option drops traffic exceeding buffer  
  With  fixed transition rate on the queue;  
      `buffer-size (MB) = buffer-size(s) * transmit-rate(Mbps) / 8`  
  With transmission rate specified as percentage;  
       `buffer-size (MB) = buffer-size (s) x interface-bandwidth (Mbps) x transmit-rate (%) / 8`

-	Remainder. This option assigns to the queue the remaining available buffer on the interface

During configuration the best practice is to match buffer-size and transmission-rate values 

#### RED drop profiles 
Random early detection declares how to drop traffic during congestion, this affects packets in buffer essentially determining how to selectively drop packets as delay buffers fill, using PLP values. Drop profiles alter buffer-size when drop probability is 100% while fill level is less than 100%, meaning can influence buffer to drop packets before it gets filled 100% hence can reduces effective maximum buffer size.
This mechanism allows us to avoid the buffer from filling up to capacity, when the buffer is full not only all the new packets get dropped but also causes global synchronization caused by TCP slow start. TCP slow start causes inefficient use of bandwidth and in order to prevent this we discard traffic after a certain threshold is reached thus throwing away some packets so that other packets don’t suffer global synchronization. “Sacrificing a few for the good of the many”
Drop profiles can be configured with below threshold mapping methods  
-	Segmented method: creates threshold map based on configured values (default method)
-	Interpolate method: creates threshold map automatically (generating 64 data points) influenced by some configured values
 
Drop profile maps are used to associate drop profiles with a scheduler. They map PLP-marked packets to drop profiles.

### Configuring scheduling
Configure drop profile
<pre>
paul@big# show class-of-service drop-profiles     
MY-PROFILE {
    interpolate {
        fill-level [ 50 70 ];
        drop-probability [ 0 10 ];
    }
}
</pre>

Configure scheduler
<pre>
paul@big# show class-of-service schedulers   
EF-SCHEDULER {
    /* traffic exceeding 15% of interface speed is buffered */
    transmit-rate {
        percent 15;
        exact;
    }
    /* when buffer (5k x 1us x 15% x interface speed) is full traffic is dropped */
    buffer-size temporal 5k;
    priority high;
}
AF-SCHEDULER {
    /* traffic exceeding 20% can be transmitted if unused bandwidth exists */
    transmit-rate percent 20;
    /* buffer is 20% of interface's available buffer and can expand dynamically */
    buffer-size percent 20;
    priority medium-low;
}
BE-SCHEDULER {
    transmit-rate {
        remainder;
    }
    buffer-size {
        remainder;
    }
    priority low;
    drop-profile-map loss-priority any protocol any drop-profile MY-PROFILE;
}
</pre>

Configure scheduler map
<pre>
paul@big# show class-of-service scheduler-maps 
SCHEDULER-MAP-CORE {
    forwarding-class AF-CLASS scheduler AF-SCHEDULER;
    forwarding-class BE-CLASS scheduler BE-SCHEDULER;
    forwarding-class EF-CLASS scheduler EF-SCHEDULER;
}
</pre>


Applying scheduler map to interface
<pre>
paul@big# show class-of-service interfaces 
ge-* {
    scheduler-map SCHEDULER-MAP-CORE;
}
</pre>

Applying scheduler map to logical interface
<pre>
paul@big# show class-of-service interfaces 
ge-0/0/0 {
    unit 0 {
        scheduler-map SCHEDULER-MAP-CORE;

paul@big# show interfaces ge-0/0/0 
per-unit-scheduler;
unit 0;
</pre>

Verification
<pre>
root# run show class-of-service scheduler-map SCHEDULER-MAP-CORE    
Scheduler map: SCHEDULER-MAP-CORE, Index: 6015

  Scheduler: BE-SCHEDULER, Forwarding class: BE-CLASS, Index: 59266
    Transmit rate: remainder, Rate Limit: none, Buffer size: remainder,
    Buffer Limit: none, Priority: low
    Excess Priority: unspecified
    Drop profiles:
      Loss priority   Protocol    Index    Name
      Low             any         13743    MY-PROFILE          
      Medium low      any         13743    MY-PROFILE
      Medium high     any         13743    MY-PROFILE
      High            any         13743    MY-PROFILE

  Scheduler: EF-SCHEDULER, Forwarding class: EF-CLASS, Index: 58382
    Transmit rate: 15 percent, Rate Limit: exact, Buffer size: 5000 us,
    Buffer Limit: exact, Priority: high
    Excess Priority: unspecified
    Drop profiles:
      Loss priority   Protocol    Index    Name
      Low             any             1    <default-drop-profile>      
      Medium low      any             1    <default-drop-profile>            
      Medium high     any             1    <default-drop-profile>      
      High            any             1    <default-drop-profile>   
{Output omitted}      
</pre>


## Rewrite Markers
This is how the router tells the next router how to process the incoming packets. We configure rewrite rules to alter CoS values in outgoing packets on the egress interfaces of an edge router to meet the policies of the next router in the path. Rewrite rule checks the current forwarding class and PLP of a packet, perform a lookup in the rewrite table and apply that CoS value to the packet leaving the router thus Conveying a packet's CoS profile to the next-hop device.
This allows the downstream routing device in a neighboring network to classify each packet into the appropriate service group. In effect, the rewrite rule performs the opposite function of the behavior aggregate (BA) classifier used when the packet enters the routing device.

Supported rewrite markings  
- ipv4 dscp
- ipv6 dscp
- ip precedence bits
- MPLS EXP bits
- 802.1p CoS bits
- 802.1ad Drop Eligible Indicator (DEI) bit

For Multiprotocol Label Switching (MPLS) EXP and IEEE 802.1 rewrite markers, values are derived from the forwarding class and PLP values in rewrite rules. MPLS EXP and IEEE 802.1 markers are not preserved because they are part of the Layer 2 encapsulation.
For IP precedence and DiffServ code point (DSCP) rewrite markers, the marker alters the first three bits on the type-of-service (ToS) byte while leaving the last three bits unchanged. 
Same forwarding-class and PLP can map to different CoS values. There is a default rewrite-rule applied when mpls is enabled on interface

Configuration
<pre>
paul@big# show class-of-service rewrite-rules 
ieee-802.1 PRI {
    forwarding-class expedited-forwarding {
        loss-priority low code-point 110;
        loss-priority high code-point 110;
        loss-priority medium-high code-point 110;
        loss-priority medium-low code-point 110;
    }
}
paul@big# show class-of-service interfaces 
ge-0/0/0 {
    unit 1000 {
        rewrite-rules {
            ieee-802.1 PRI;
        }
    }
}
</pre>

Verfication
<pre>
paul@big# run show class-of-service rewrite-rule name PRI 
Rewrite rule: PRI, Code point type: ieee-802.1, Index: 5137
  Forwarding class                    Loss priority       Code point
  expedited-forwarding                low                 110            
  expedited-forwarding                high                110            
  expedited-forwarding                medium-low          110            
  expedited-forwarding                medium-high         110            

paul@big# run show class-of-service interface ge-0/0/0.1000 
  Logical interface: ge-0/0/0.1000, Index: 69
    Object                  Name                   Type                    Index
    Rewrite                 PRI                    ieee8021p (outer)        5137
</pre>


## Summary with network scenario and configuration

 
Fig: MPLS PE-CE network

In this scenario the MPLS core network is configured with 4 queues with markings as described in the below table.

+---------------------+-------+---------------+----------------+  
|CLASS OF SERVICE	    | QUEUE | LOSS PRIORITY	|EXP COS MARKING |  
+---------------------+-------+---------------+----------------+  
|Expedited Forwarding	| 0   	| Low           |	101 (5)        |  
+---------------------+-------+---------------+----------------+  
|Assured Forwarding   |	1     |	Low	          | 100 (4)        |  
+---------------------+-------+---------------+----------------+  
|Network Control	    | 2     |	Low           |	110 (6)        |  
+---------------------+-------+---------------+----------------+  
|Best Effort          |	3     |	Low           |	000 (0)        |  
+---------------------+-------+---------------+----------------+  

Table: MPLS EXP COS Markings

> Note: All of the configuration displayed below are under the class-of-service stanza  

CLASSESS & CLASSIFIERS: MAP TRAFFIC TO QUEUES  
This configuration is done in all routers in the provider network
<pre>
forwarding-classes {
     class EF queue-num 0 priority low;
     class AF queue-num 1 priority low;
     class NC queue-num 2 priority low;
     class BE queue-num 3 priority low;
}
classifiers {
     exp MPLS-In {
         import default;
         forwarding-class EF {
             loss-priority low code-points 101;
         }
         forwarding-class AF {
             loss-priority low code-points 100;
         }
         forwarding-class NC {
             loss-priority low code-points 110;
         }
         forwarding-class BE {
             loss-priority low code-points 000;
         }
     }
     ieee-802.1 LAYER2-In {
         import default;
         forwarding-class EF {
             loss-priority low code-points 101;
         }
         forwarding-class AF {
             loss-priority low code-points 100;
         }
         forwarding-class NC {
             loss-priority low code-points 110;
         }
         forwarding-class BE {
             loss-priority low code-points 000;
         }
     }
}
</pre>

SCHEDULER & SCHEDULER-MAPS: TREATMENT OF PACKETS DURING CONGESTION ON EGRESS  
This configuration is done in all routers in the provider network
<pre>
schedulers {
     EF-scheduler {
         transmit-rate percent 15 exact;
         buffer-size temporal 5k;
         priority high;
     }
     AF-scheduler {
         transmit-rate percent 20;
         buffer-size percent 20;
         priority medium-low;
     }
     NC-scheduler {
         transmit-rate percent 5;
         buffer-size percent 5;
         priority medium-low;
     }
     BE-scheduler {
         transmit-rate remainder;
         buffer-size remainder;
         priority medium-low;
         drop-profile-map loss-priority low protocol any drop-profile DP-aggressive;
     }
}
drop-profiles {
     DP-aggressive {
         interpolate {
             fill-level [ 50 70 ];
             drop-probability [ 0 10 ];
         }
     }
}
scheduler-maps {
     scheduler-map-core {
         forwarding-class EF scheduler EF-scheduler;
         forwarding-class AF scheduler AF-scheduler;
         forwarding-class NC scheduler NC-scheduler;
         forwarding-class BE scheduler BE-scheduler;
     }
}
</pre>

REWRITE RULES: CONVEYING COS TO DOWNSTREAM ROUTERS  
This configuration is done in all routers in the provider network
<pre>
rewrite-rules {
    exp MPLS-out {
        import default;
        forwarding-class EF {
            loss-priority low code-point 101;
        }
        forwarding-class AF {
            loss-priority low code-point 100;
        }
        forwarding-class NC {
            loss-priority low code-point 110;
        }
        forwarding-class BE {
            loss-priority low code-point 000;
        }
    }
}
</pre>

CORE FACING INTERFACES: PE & P ROUTER'S CORE FACING MPLS INTERFACES
<pre>
interfaces {
    $interface_identifier$ {
        unit 0 {
            scheduler-map scheduler-map-core;
            classifiers {
                exp MPLS-In;
            }
            rewrite-rules {
                exp EXP-out;
            }
        }
    }
}
</pre>

CUSTOMER FACING INTERFACES
This configuration is done on PE routers with customers requiring QoS. To be configured using Fixed, BA or MF classifiers and mapped to forwarding classes as see fit. Below example uses fixed classification to put all the traffic from CE to the assured forwarding class.
<pre>
interfaces {                            
    ge-0/0/1 {
        unit 100 {
            forwarding-class AF-CLASS;  
       }
   }
}
</pre>

