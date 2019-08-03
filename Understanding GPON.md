# GPON Overview

A PON is a fiber network that only uses fiber and passive components like splitters and combiners rather than active components like amplifiers, repeaters, or shaping circuits. Such networks cost significantly less than those using active components.
The main disadvantage is a shorter range of coverage limited by signal strength. While an active optical network (AON) can cover a range to about 100 km (62 miles), a PON is typically limited to fiber cable runs of up to 20 km (12 miles). PONs also are called fiber to the home (FTTH) networks.
GPON is a Gigabit PON, Gigabit Passive Optical Network, it derives its name from the speed it provides. It delivers 2.488 Gbits/s downstream and 1.244 Gbits/s upstream.

## Merits of GPON network  
•	Longer transmission distance:  resolving transmission distance and bandwidth issues in twisted pair transmission.   
•	Higher bandwidth: Each GPON port supports a maximum transmission rate 2.5 Gbit/s downstream and 1.25 Gbit/s upstream  
•	Higher resource usage with lower costs: GPON supports a split ratio up to 1:128. A feeder fiber from the CO equipment room can be split to up to 128 drop fibers. This economizes on fiber resources and O&M cost  

## GPON in comparison to other optical access network deployments
A passive optical network, does not include electrically powered switching equipment and instead uses optical splitters to separate and collect optical signals as they move through the network. A passive optical network shares fiber optic strands for portions of the network. Powered equipment is required only at the source and receiving ends of the signal.
An active optical system uses electrically powered switching equipment, such as a router or a switch aggregator, to manage signal distribution and direct signals to specific customers. In such a system, one optical fiber connects to a switch. The switch rebroadcasts the data to each individual user. A separate cable is routed from the switch to each individual user.
See figure below for physical topology comparison highlighting the pros of a PON network.  

![opticalnet](https://user-images.githubusercontent.com/50369643/62413491-1b991b80-b618-11e9-86fa-1503022b8135.png)

## GPON Transmission
GPON is a point to multi-point (P2MP) passive optical network. GPON uses optical wavelength division multiplexing (WDM) so a single fiber can be used for both downstream and upstream data. A laser on a wavelength of 1490 nm transmits downstream data. Upstream data transmits on a wavelength of 1310 nm.  
GPON Working principle: A GPON network is composed of OLT, ODN and ONU.  

![olt-onu](https://user-images.githubusercontent.com/50369643/62413515-7894d180-b618-11e9-83dd-2242ac47299a.png)


### GPON System components
1.	Optical Line terminal (OLT): is an aggregation device located at the central office. It typically provides Ethernet uplinks to the internet (WAN)
2.	Optical Network Units (ONUs)/Optical Network Terminal (ONTs): are located on the user side, providing various ports for connecting to user terminals. These typically have LAN Ethernet ports
3.	The Optical distribution Network (ODN):  is the physical fiber infrastructure connecting the OLT to the ONTs. It consists of a tree of fiber cables and one or more optical splitters. 

![ont-olt](https://user-images.githubusercontent.com/50369643/62413534-c90c2f00-b618-11e9-99db-6515daecf059.png)

![oo](https://user-images.githubusercontent.com/50369643/62413548-e3460d00-b618-11e9-9709-826687ba0787.png)

Service Node Interface (SNI) is the Ethernet uplink of the OLT and this is usually connected to an Ethernet switch or router in the service provider network. The User to Network Interface (UNI) interfaces the customer equipment to the PON network using Ethernet LAN connection. In common deployment the ONT is only bridging the CPE to PON network, nevertheless some advanced ONU models can also operate on router mode, providing connection to hosts.

The internal configuration combining OLT and ONT can be conceptualized as depicted in the following sketch
![gponsketch](https://user-images.githubusercontent.com/50369643/62413571-125c7e80-b619-11e9-9e39-ad7a25630e70.png)

### GPON Service multiplexing
GPON adopts two multiplexing mechanisms for transmission  
#### (a)	Downstream direction (i.e. from OLT to users)  
  All data is broadcast to all ONUs from the OLT. The ONUs then select and receive their respective data and discard the other data. An ONU identifies the desired data by ONU ID (GEM port ID)

![gponmux](https://user-images.githubusercontent.com/50369643/62413591-4637a400-b619-11e9-895c-ba761a47ad07.png)

#### (b)	Upstream direction (i.e. from users to OLT)  
  Data packets are transmitted in a Time Division Multiple Access (TDMA) manner based on time slots
Each ONU can send data to the OLT only in the timeslot permitted and allocated by the OLT. This ensures that each ONU sends data in a given sequence, avoiding upstream data conflicts. The Detects and prevents collisions through ranging

![gpondemux](https://user-images.githubusercontent.com/50369643/62413606-6bc4ad80-b619-11e9-8178-6589f245b68e.png)

In Ranging the OLT measures delay and sets a register in each ONU via PLOAM (Physical Layer Operations, Administration and Maintenance) messages to equalize its delay with respect to all other ONUs on the access network
Once the delay of all ONUs have been set, the OLT transmits grants to individual ONUs. A grant is permission to use a defined interval of time for upstream transmission. The grant map is dynamically re-calculated every few milliseconds. The map allocates bandwidth to all ONUs such that each ONU receives timely bandwidth for its needs.

## GPON Encapsulation
GPON makes use of GEM Ports and T-CONTs. GEM ports and T-CONTs divide a PON network into virtual connections for service multiplexing

**GEM:**  
> Refers to GPON Encapsulation Method, and is the native protocol used for transmitting data within a GPON.

**T-CONT:**  
> A Transmission Container is an ONU object representing a group of logical connections that appear as a single entity for the purpose of upstream bandwidth assignment on the PON. All GEM ports are mapped to T-CONTs. Then service streams are transmitted upstream by means of OLT's dynamic bandwidth allocation (DBA) scheduling
A T-CONT can be bound to one or more GEM ports, depending on customers' data plan. On the OLT, GEM ports are demodulated from the T-CONT and then service streams are demodulated from the GEM port payload for further processing

**GEM Port:** 
> Is a virtual port for transmitting frames between the OLT and the ONU/ONT and can be described as a logical point-to-point connection between OLT and ONT. 
Each GEM port can carry one or more types of service stream. After carrying service streams, a GEM port must be mapped to a T-CONT before upstream service scheduling. Each ONU supports multiple T-CONTs that can have different service types.
Several GEM Ports can be assigned to a T-CONT, one GEM port can carry up to 8 GEM connections.

![tcont](https://user-images.githubusercontent.com/50369643/62413621-a29ac380-b619-11e9-9d8c-7d73fe357e36.png)

**DBA:**
> Dynamic Bandwidth Allocation (DBA) is a methodology that allows adoption of users' bandwidth allocation based on current traffic requirements.
It allows upstream timeslots to shrink and grow based on the distribution of upstream traffic loads. DBA functions on T-CONTs and provides a way for ONT and OLT to communicate on bandwidth requirement, OLT polls the ONU which in turn responds with a report of data currently waiting to be sent (traffic in T-CONT buffer) OLT receives the status report and sends bandwidth-allocation to the ONU.
When ONT has nothing to send, it sends an idle cell to the OLT which in turn allocate the bandwidth to other T-CONTs

## Service mapping relationship
#### (a)	In the upstream direction:  
ONU sends Ethernet frames to GEM ports based on configured mapping rules between service ports and GEM ports. Then, the GEM ports encapsulate the Ethernet frames into GEM packet data units and add the PDUs to T-CONT queues based on the configured mapping rules between GEM ports and T-CONT queues. Then, the T-CONT queues use timeslots for upstream transmission to send GEM PDUs to the OLT.
The OLT receives the GEM PDUs and obtains Ethernet frames from them. Then, the OLT sends Ethernet frames from a specified uplink port based on the configured mapping rules between service ports and uplink ports.
For the upstream from ONU to OLT, a time division multiplex (TDM) technique is used where each user is assigned a timeslot on a different wavelength of light. Because the TDM method involves multiple users on a single transmission, The OLT determines the distance and time delay of each subscriber. Then software provides a way to allocate timeslots to upstream data for each user. This advocates the fact that the upstream rate is less than the maximum because it is shared with other ONUs in a TDMA scheme. 

![gemport](https://user-images.githubusercontent.com/50369643/62413638-dc6bca00-b619-11e9-8599-2490e86ce208.png)


#### (b)	In the downstream direction:  
The OLT sends Ethernet frames to the GPON service processing module based on configured mapping rules between service ports and uplink ports. The GPON service processing module then encapsulates the Ethernet frames into GEM PDUs for downstream transmission using a GPON port. 
GPON transmission convergence (GTC) frames containing GEM PDUs are broadcast to all ONUs connected to the GPON port. 
The ONU filters the received data according to the GEM port ID contained in the GEM PDU header and retains the data only belonging to the GEM ports of this ONU. Then, the ONU decapsulates the data to Ethernet frames and sends them to end users using service ports.
In the basic method of operation for downstream distribution on one wavelength of light from OLT to ONU/ONT, all customers receive the same data. The ONU recognizes data targeted at each user.

![gemport2](https://user-images.githubusercontent.com/50369643/62413648-10df8600-b61a-11e9-98e5-802bdb52f918.png)

# Service provisioning on Huawei OLT MA56xx 
## Configuration via U2000 NMS

#### 1. Create Traffic profile: 
This defines Committed Information Rate CIR and Peak Information Rate PIR to be applied on ONT, it is used on the ONT to control upstream and downstream traffic.  
Procedure:  
(a) Choose Configuration > Access Profile Management > Traffic Profile  
(b) Click the MEF IP Traffic Profile tab.  
(c) Right-click and choose Add Global Profile.  
(d) In the dialog box set the  
> [ profile name | CIR | optional PIR|CBS|PBS] 
> CIR=Committed Information Rate  
> CBS=Committed Burst Size  
> PIR=Peak Information Rate  
> PBS=Peak Burst Size  

(e) Click OK  
(f) In the MEF IP Traffic profile list, right-click the profile created and Download to NE  
(g) Select the required NE(s) you want the profile to be used in, and click OK  


#### 2. Create DBA profile:  
This defines bandwidth to be applied on Line profile for upstream traffic  
Procedure:  
(a) Choose Configuration > Access Profile Management > GPON Profile  
(b) Click the DBA Profile tab.  
(c) Right-click and choose Add Global Profile.  
(d) In the dialog box set the parameters  
> [ Name | TCONT (DBA) type and consequently its bandwidth]  

(e) Click OK  
(f) In the DBA profile list, right-click the profile and choose Download to NE  
(g) Select the required NE(s) you want the profile to be used in  
(h) and click OK  


#### 3. Create Line profile:  
The line profile binds the DBA to a T-CONT, specifies GEM ports and GEM connections with their respective VLANs. One or multiple (up to 8) GEM connections (each with different VLAN) can be added on a single GEM port. In a GEM port, different GEM connections need to be set up for different service streams.  
Procedure:  
(a) Choose Configuration > Access Profile Management > GPON Profile   
(b) Click the Line Profile tab.  
(c) Right-click and choose Add Global Profile.  
(d) In the dialog box set the Name  
(e) Click on Base Info and set the parameters  
> [ Mapping Mode: VLAN | QoS Mode: Priority Queue]  

(f) Click on Line then right-click T-CONT Info and choose ADD TCONT from the shortcut menu.  
(g) In the dialog box set the parameters  
> [–TCONT Index | –Assign DBA Profile]  

(h) Right-click the created T-CONT and choose Add GEM Port  
(i) In the dialog box set the parameters
> [–GEM Port Index] and click OK  

(j) Right-click the created GEM Port and choose Add GEM Connection  
(k) In the dialog box set the parameters
> [–GEM Connection Index |–VLAN ID]  

(l) In the line profile list, right-click the profile and choose Download to NE.  
(m) Select the required NE(s) you want the profile to be used in, and click OK  


#### 4. Create Service profile:  
This should be consistent with the actual ONT type as it depends on port density and port type of the ONT  
Procedures:  
(a) Choose Configuration > Access Profile Management > GPON Profile from the main menu.  
(b) Click the Service Profile tab.  
(c) Right-click and choose Add Global Profile from the shortcut menu.  
(d) In the dialog box set the Name  
(e) Choose Base Info and set the parameters  
> [–Number of Pots Ports |–Number of ETH Ports]  

(f) Choose UNI Port and right-click the record where Port Type is set to ETH and Port ID is set to 1 and choose UNI Port Configuration Properties  
(g) In the dialog box that is displayed, right-click and choose Add, and configure the parameters of VLAN switch.  
(h) Set 
> [–Service Type: Translation |–SVLAN |–CVLAN (Client VLAN)] and Click OK.  

(i) In the service profile list, right-click the profile and choose Download to NE.   
(j) Select the required NE(s) you want the profile to be used in, and click OK   
	
#### 5. Register the ONT:  
This process involved adding the ONU/ONT to the OLT PON port  
Procedure:  
(a) In the Main Topology, double-click or right-click the required OLT and choose NE Explorer  
(b) Choose GPON then GPON UNI Port Tab  
(c) On the GPON UNI Port tab page, set the filter criteria to display the required GPON UNI ports.   
(d) From the list, right-click GPON UNI port where the ONT is connected and Enable ONU Auto Find.  
(e) Click the Auto-Discovered ONU Info tab in the lower pane, click the ONT and then choose Confirm ONU.  
(f) Set parameters  
> [–Name |–ONU Type: ONT]  

(g) On the Basic Parameters tab set the parameters  
> [–Line Profile |–Service Profile|–Authentication Mode: SN |–Terminal Type]  

(h) Click OK  

#### 6. Configure the service:  
Procedure:  
6.1. Configuring the ETH Port of a GPON ONU as tagged or untagged  
(a)	Choose GPON > GPON ONU from the navigation tree.  
(b)	On the GPON ONU tab page, set the filter criteria to display the required GPON ONUs.  
(c)	Select the ONT from the list and click the ONT's UNI Port Info tab in the lower pane.   
(d)	On the ONT's UNI Port Info tab page, right-click the record where UNI Type is set to ETH and UNI ID is set to 1, and choose Modify.  
(e)	In the dialog box set Default VLAN ID. The default VLAN will be untagged, any other VLAN on this port will be tagged  
(f)	Click OK.  

6.2. Configure a service VLAN on the OLT side. A service VLAN is the VLAN used for the service.  
(a)	Choose VLAN from the navigation tree.  
(b)	On the VLAN tab page right-click and choose Add.  
(c)	In the dialog box set the parameters  
> [–VLAN ID |–Type: Smart VLAN]  

(d)	Click Next. Click the Upstream Port tab and add upstream port of the VLAN.  
(e)	Click Done  

6.3. Add a service virtual port on the OLT side.  
(a)	On the VLAN tab page, select the record with the VLAN ID and click the Service Port tab in the lower pane.  
(b)	From the service port tab, right-click and choose Add.  
(c)	In the dialog box set the parameters:  
> [– Name |–VLAN ID (SVLAN) |–Connection Type: LAN-GPON |–Interface Selection (choose the GEM Port) |–Service Type: Multi-Service VLAN |– User VLAN (VLAN) |–Keep the upstream and downstream settings the same: selected]  


## Procedures in summary

<img width="262" alt="gponprovision" src="https://user-images.githubusercontent.com/50369643/62414231-f9f06200-b620-11e9-8ea1-a57449ae713a.PNG">

## Configuration via CLI

### Auto discover ONT
 `dispaly ont autofind all`

### Create Service-profile
<pre>
 ont-srvprofile gpon profile-name "SERV_PRF"
  ont-port pots 2 eth 4 
  transparent enable 
  port vlan eth 1 translation 101 user-vlan 101
  port vlan eth 1 translation 102 user-vlan 102
  port vlan eth 1 translation 103 user-vlan 103
  port vlan eth 1 translation 104 user-vlan 104
  port vlan eth 1 translation 105 user-vlan 105
  port vlan eth 2 translation 106 user-vlan 106
  port vlan eth 3 translation 107 user-vlan 107
  port vlan eth 4 translation 108 user-vlan 108
  commit
</pre>

### Create DBA profile
<pre>
 dba-profile add profile-name "CIR_160Mb" type1 fix 163840 bandwidth_compensate no
</pre>

### Create Line-profile
<pre>
 ont-lineprofile gpon profile-name "LINE_PRF"
  tcont 3 dba-profile-name CIR_160 
  gem add 0 eth tcont 3 | Add GEM-Port 0 to T-CONT 3
  gem add 1 eth tcont 3
  gem add 2 eth tcont 3
  gem add 3 eth tcont 3
  gem add 4 eth tcont 3
  gem add 5 eth tcont 3
  gem add 6 eth tcont 3
  gem add 7 eth tcont 3
  gem mapping 0 0 vlan 101 | on GEM-Port 0 add Gem-connection 0 with CVLAN 101
  gem mapping 1 1 vlan 102 | on GEM-Port 1 add Gem-connection 1 with CVLAN 102
  gem mapping 2 2 vlan 103
  gem mapping 3 3 vlan 104
  gem mapping 4 4 vlan 105
  gem mapping 5 5 vlan 106
  gem mapping 6 6 vlan 107
  gem mapping 7 7 vlan 108
  commit
  quit
</pre>

### Create traffic profile to be used on service-port
 `traffic table ip name "160Mb" cir 163840 cbs 5242880 pir 158720 pbs 5242880 priority 0 priority-policy local-setting`

### Create SVLAN
 `vlan desc 326 description "CUSTOMER1"`

### Add the SVLAN to the ethernet port to PE
The OLT port connecting to PE is 0/7/0  
 `port vlan 234 0/7 0 `
 

### Add ONT
Frame 0 Slot 8 Port 0 ONT-ID 0
<pre>
 interface gpon 0/8
 ont add 0 0 sn-auth "4857544308D8DD10" omci ont-lineprofile-name LINE_PRF ont-srvprofile-name SRV_PRF desc "SITE1" 
</pre>

## Create service-port
Provision ONT on PON Port 0/8/0 ONT-ID 0
<pre>
 service-port 627 vlan 326 gpon 0/8/0 ont 0 gemport 0 multi-service user-vlan 101 tag-transform translate inbound traffic-table name 160Mb outbound traffic-table name 160Mb
 service-port 628 vlan 327 gpon 0/8/0 ont 0 gemport 1 multi-service user-vlan 102 tag-transform translate inbound traffic-table index 160Mb outbound traffic-table name 160Mb
</pre>

