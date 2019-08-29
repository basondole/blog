# Switch Port Analyzer
Switched Port Analyzer (SPAN), sometimes called port mirroring or port monitoring, copies switch network traffic and forwards it out the SPAN port for analysis by a network analyzer. By enabling the SPAN, you can monitor traffic on a switch port by forwarding incoming and outgoing traffic to another port for data collection and analysis. You can use a network analyzer on this monitor port to troubleshoot network problems by examining traffic on other ports or segments without taking the network out of service.

# Basic SPAN configuration 
Basic SPAN captures traffic from one source port or VLAN and sends the traffic to another port on the same switch. On the diagram shown below, we will capture traffic from source port fa0/1 (connected to a user computer) and send the traffic to destination port fa0/5

![span](https://user-images.githubusercontent.com/50369643/63925570-fbac1a80-ca52-11e9-9fd0-15fe4fb22ccb.png)

## Configuration Commands
<pre>
Switch# configure terminal
Switch(config)# monitor session 1 source interface fa0/1
Switch(config)# monitor session 1 destination interface fa0/5
</pre>

With this simple configuration, traffic sourced from interface fa0/1 will be mirrored to interface fa0/5 where you will be able to capture it.


## Basic RSPAN configuration
RSPAN (Remote SPAN) feature allows traffic that is sourced from a switch to be mirrored to a remote switch within a layer 2 network over trunk ports. To accomplish this you will have to configure the destination VLAN across the entire path between the switches.
In the diagram below, we want to capture traffic from Switch1 (port fa0/1) and send the traffic to Switch2 (port fa0/5).  
Although here we show a direct connection with a Layer2 trunk port between Switch1-Switch2, you can have multiple switches between them with no problem (the capturing vlan must be active on the whole path though).

![rspan](https://user-images.githubusercontent.com/50369643/63930010-269a6c80-ca5b-11e9-8244-6424de8638a4.png)

### Configuration Commands
<pre>
Switch1# config term
Switch1 (config)# vlan 100
Switch1 (config-vlan)# remote span   
Switch1(config-vlan)# exit
Switch1 (config)# monitor session 10 source interface fa0/1
Switch1 (config)# monitor session 10 destination remote vlan 100

Switch2_Remote# config term
Switch2_Remote (config)# vlan 100
Switch2_Remote (config-vlan)# remote span
Switch2_Remote (config-vlan)# exit
Switch2_Remote (config)# monitor session 11 source remote vlan 100
Switch2_Remote (config)# monitor session 11 destination interface fa0/5
</pre>


All traffic sourced from interface fa0/1 on switch 1 will be forwarded using vlan 100 towards the destination port on remote switch2 where you can sniff the traffic.
## Basic ERSPAN configuration  
ERSPAN (Encapsulated Remote Switched Port Analyzer) is a feature present on the new IOS-XE on ASR1000 but is also available on Catalyst 6500 or 7600. It is used to send traffic for sniffing over layer3 networks and it works by encapsulating the traffic using a GRE tunnel.
On the diagram below, there is a GRE tunnel between Switch1 (again this is usually an ASR1000 or 7600 etc) and remote Switch2. The GRE tunnel is established between IP address 172.16.10.10 (on switch1) and 10.10.10.10 (on switch2). We want to send traffic from fa0/1 on Switch1 to fa0/5 on Switch2.

![erspan](https://user-images.githubusercontent.com/50369643/63930622-5c8c2080-ca5c-11e9-8a67-d2e5398b3934.png)

Configuration Commands
The erspan-id must match on both devices

<pre>
Switch1(config)# monitor session 1 type erspan-source
Switch1 (config-mon-erspan-src)# source interface fa0/1
Switch1 (config-mon-erspan-src)# destination
Switch1 (config-mon-erspan-src-dst)# erspan-id 110
Switch1 (config-mon-erspan-src-dst)# ip address 10.10.10.10 # ip address on switch2
Switch1(config-mon-erspan-src-dst)# origin ip address 172.16.10.10 # local ip on switch 1

Switch2_Remote (config)# monitor session 1 type erspan-destination
Switch2_Remote (config-mon-erspan-dst)# destination interface fa0/5
Switch2_Remote (config-mon-erspan-dst)# source
Switch2_Remote (config-mon-erspan-dst-src)# erspan-id 110
Switch2_Remote (config-mon-erspan-dst-src)# ip address 10.10.10.10

</pre>

### There’s an Easier Way ...    
When configuring the IP address of the destination, you can simply enter the IP address of your own PC and all of the encapsulated mirrored traffic is sent to your PC’s IP address. With a simple capture filter setup in Wireshark you can limit your captured packets only to GRE packets. Now you’re only seeing the mirrored traffic. But it gets better. Wireshark is very smart. It realizes that the traffic is encapsulated and automatically displays the “real” source and destination IP addresses of the captured traffic, not the source switch’s IP address and your PC’s (destination) IP address. As a bonus, if you’re sniffing a VLAN trunk, the 802.1Q tags are also captured in the ERSPAN header info

## Configuration
On Cisco IOSXE
<pre>
KH16(config)#do sh run | se monitor session
monitor session 1 type erspan-source
 description ERSPAN DIRECT TO WIRESHARK
 source interface Gi3
 destination
  erspan-id 32
  mtu 1464
  ip address 192.168.56.1
  origin ip address 192.168.56.26
KH16(config)#
</pre>
Here the IP address 192.168.56.1 is the ip address of a PC running wireshark


On your Sniffer PC running Wireshark, you’ll want to configure a Capture Filter that limits the captured traffic to IP Protocol number 47, which is GRE. 47 in HEX is 2F, so the capture filter for this is ip proto 0x2f.
Click “capture” on the toolbar then choose “capture filter” a window will pop up

![wiresharkfilter](https://user-images.githubusercontent.com/50369643/63931427-ebe60380-ca5d-11e9-9ffd-eea4ceef6407.png)

Click the plus sign on the lower left and add the name and filter value

![gre-filter](https://user-images.githubusercontent.com/50369643/63931554-2ea7db80-ca5e-11e9-832e-2b510f53bec7.png)

Then start a capture on your PC interfaces on the filter box type erspan then enter to filter the packets recived from the ERSPAN source

![capturegre](https://user-images.githubusercontent.com/50369643/63931786-9a8a4400-ca5e-11e9-9384-67f272caa25f.png)
