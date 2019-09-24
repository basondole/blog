# Guide To Network Automation  

## Raw Python
Python offers support to interact with our network devices for management and administration by using different sets of libraries. On machine with python install we can use the raw python interpreter from Windows cmd or terminal for Mac and Linux to send commands and retrieve information or send configuration to devices. Python also comes with an IDE which can be used as well. 
In order to be able to interface with the devices we need to have appropriate libraries installed. Think of libraries as add-ons that adds features to the native python. Python by default ships with core modules for basic functionality however you can install additional modules to extend the functionality.
Generally we can interact with our devices by using the native CLI using telnet and SSH or by using APIs such as NETCONF and RESTCONF. Luckily there are libraries for all these methods in python.

### CLI methods
On this section we’ll cover popular libraries available in python that supports native raw interaction with the network device CLI as regular ssh or telnet clients. Some of the popular libraries include  
- Telnetlib  
- Paramiko  
- Netmiko  
- Pyntc  


These libraries allows for initialization of remote sessions to devices via SSH or telnet and sending of commands to the devices as you would via an ssh or telnet client such as putty or secureCRT.  

#### Telnetlib  
The telnetlib uses the telnet protocol to communicate to devices and send raw cli commands. The telnetlib comes with standard python installation and does not require a separate install, as soon as you got python installed on your workstation you are good to go. See demonstration below on how it works  

Pre-requisite: telnet must be enabled on the remote device.  

##### Telnetlib demo  

For the syntax in the python interpreter where each numbered action is executed jump to the following snapshot.
1.	Import the telnetlib in python interpreter  
2.	Create a function that will be used to establish connection  
3.	Create a function that will be used to send commands to the device  
4.	Define login parameters  
5.	Connect to the device  
6.	Send sample command to the device  
7.	Send another sample command to the device  
8.	End the session and neatly exist.  

> telnet images

#### Paramiko
Paramiko uses the SSH protocol to communicate with devices and sends raw cli commands as it will be shown in the demonstration. In very light language paramiko does what the telnetlib does but with SSH instead of telnet however paramiko is not a standard python library and must be installed separately.  
To install paramiko on machine with python and pip installed do: `pip install paramiko`  
Pre-requisite: 
1.	ssh is enabled on the remote device  
2.	paramiko library is installed on the local working station  

##### Paramiko demo
Again, the syntax for each of the below actions is shown in the next snapshot  
1.	Import the paramiko module  
2.	Create a paramiko ssh client object  
3.	Specify the policy for missing host keys from the ssh server  
4.	Define login parameters  
5.	Connect to the device  
6.	Invoke the device shell to interact with  
7.	Send command  
8.	Retrieve the output from the device  
9.	Send another sample command  
10.	Retrieve the output from the device  
11.	Close the session neatly  

> paramiko images

#### Genie
Genie is both a library framework and a test harness that facilitates rapid development, encourage re-usable and simplify writing test automation. Genie was initially developed internally in Cisco, and is now available to the general public. Genie offers both a python library and a cli based tool you can use directly from your command prompt (or terminal) without running the python interpreter. Learn more about Genie at https://developer.cisco.com/site/pyats/

Genie uses the idea of testbeds where basically a testbed is where you define parameters of your device such as IP address, hostnames, login credential and transport protocol. Basically it is an equivalent of ansible inventory or routers.db for rancid respectively if you have worked with those before. Ansible is gonna be covered later in this write up, for rancid learn more via https://github.com/basondole/tutorials/tree/master/rancid   

Genie supports both telnet and ssh as transport protocols the caveat is however, similar to rancid and ansible, genie does not natively run on windows and since it also a non-standard python to install genie on a non-windows machine with python and pip installed do: `pip install genie`

Sample genie testbed

>genie snaps


Important: the hostname used in the testbed must be the exact match with the device configured hostname. I had so much problems before figuring this out :)

##### Genie demo
In this demo we are going to extract interface information from the device and record in a csv file the interface names and their mac addresses  
For the syntax in the python interpreter where each numbered action is executed jump to the snapshot thereafter.  
First I switchover to a linux box (wsl and run ipython) since genie is not supported on windows  
1.	Import the genie module  
2.	Load the testbed file  
3.	Verify which devices are in the testbed and we only have KH16 in our testbed  
4.	Create an object for the device  
5.	Use the object to connect to the device. This will display the commands that have been issued by genie to the device which I will cut out of the snapshot  

> genie snap 1

6.	Verify the connection  
7.	Get interface information from device. This will display the commands that have been issued by genie to the device together with the output which I will cut out of the snapshot  

> genie snap 2

8.	Import the csv module for creating a csv file  
9.	Define the field of the csv file  
10.	Write the data to the csv file  
11.	Close the session  
12.	Exit and verify the contents of the file created on step 10 above  

> genie snap 3

We can verify the new file is created as interfaces.csv and we can verify the contents

#### Netmiko
Netmiko is yet another option. Essentially it is a multi-vendor library to simplify Paramiko SSH connections to network devices. Learn more about Netmiko via https://github.com/ktbyers/netmiko

#### Pyntc
Pyntc is a simple open source multi-vendor Python library that establishes a common framework for working with different network APIs & device types. Checkout this self-explanatory repo to learn more about Pyntc https://github.com/pyntc


#### Summary
In this section we have covered the python libraries that allow us to use conventional CLI commands to interact with the devices. The CLI methods allows user to send raw commands to devices via SSH or telnet as they would on the regular CLI, some libraries offer pre-defined functions that can be used to perform specific tasks an example will be a method that gets the running-config or reboot the device without the user issuing the specific CLI commands and rather just call the method, the commands are generated by the function itself and are sent via SSH to the CLI for execution.


### API & Remote Procedure Calls method
In this section we cover the basics on interaction with network devices using APIs. This is however only useful when the device supports API interactions. The most popular protocol in this category is NETCONF. NETCONF is a protocol defined by the IETF to install manipulate and delete configuration of network devices. NETCONF operations are realized on top of a Remote Procedure Call using XML encoding. Equivalent protocols include RESTCONF and gRPC all of which make use of the YANG models.  

The easiest way to understand this is to think of YANG as an SNMP MIB where as NETCONF and RESTCONF are like SNMP versions. They are responsible to connect to the devices, package and send requests to retrieve info or send configuration details.  

YANG is really a data structure to describe properties of an element using models (modules) analogous to SNMP MIB. There are different flavors of data models, these model include:  
- IETF Models: IETF issues the industry standard models  
- OpenConfig Models: OpenConfig is a consortium of vendors that issues YANG models  
- Native Models: Models developed by specific vendors such as Cisco or Juniper  

Learn more about YANG at https://github.com/YangModels/yang  
NETCONF runs over SSH tcp port 830 where as RESTCONF runs over HTTPS leveraging REST API standards. gRPC is created by google aiming to interface with everything in their data centres from switch, router and servers.


### NETCONF
#### Configuring NETCONF on the device

##### Cisco IOSXE
Issue the command netconf-yang in configuration mode
> ios yang snap

##### JunOS

> junos yang snap

To verify the device is listening to NETCONF we open a netconf session via ssh specifying the netconf port (-p 830) and the service (-s netconf)

> snap

The session will open and the device will send its capability list in what is called a hello message, which is essentially the models it supports. To complete the hello we have to send our computers capability back to the device in an XML format  

```
<?xml version="1.0" encoding="UTF-8"?>
   <hello xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
      <capabilities>
         <capability>urn:ietf:params:netconf:base:1.0</capability>
      </capabilities>
   </hello>]]>]]>
```
Now the hello process is completes we can interact with the device by sending XML commands packaged as RPC (remote procedure calls) which tell the device what to do. However for now we will just issue an XML command that ends the session neatly without leaving a hanging vty session on the router.

```
<?xml version="1.0" encoding="UTF-8"?>
   <rpc message-id="1239123" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
       <close-session />
    </rpc> ]]>]]>
```

> snap

As you can see it is not very user friendly as the regular CLI, this is because NETCONF is intended for writing scripts & network management tools. Mostly we work with NETCONF using a programming language such as python. The value for NETCONF is not replacing the CLI for running one command or one operational task. The value for NETCONF is doing this type of configuration across huge networks of devices not just one at a time.  

#### NETCONF with python  
Pre-requisite:  
-	NETCONF is enabled on the network device
-	Python and ncclient are installed on the workstation

##### NETCONF Cisco demo
On this demo we use netconf to get config from the device and also to push config to the device running Cisco IOSXE  
First we launch the python interpreter. I’m using ipython  
1.	Import the manager module from the netconfclient (ncclient library)  
Your workstation is the manager whereas the network device is running the agent code  
2.	Define the router and its login parameters  
3.	Connect to the router using the netconfclient manager  
4.	Verify the device is connected  
5.	Crete an xml filter to get config of interface GigabitEthernet3 since netconf uses xml format  
6.	Get the running config from the device and apply the filter from step 5 to get only Gigethernet3 config  
7.	Verify the operation was successful  
8.	Import a library that will help us convert xml data returned by netconf to a python dictionary which is easier to navigate  
9.	Convert the xml data collected at step 6 to a python dictionary  
10.	To extract the actual data, since netconf returns structured data we have to navigate to specific blocks of information in this case we go to the ‘rpc-reply’ section then dive into the ‘data’ section which is a leaf of the ‘rpc-reply’  
11.	Use the dictionary keys (which I already know because I know the data structure) to access the data of interest in this case we are extracting the interface name. In this case we get the return value datatype (OrderedDict) the data model (ietf…) and the #text which is the name of the interface  
12.	Get the ipv4 address configured on the interface, we follow same paradigm as above except we want the key ‘ipv4’ instead of ‘name’  
13.	To send config to the device we create an xml block of our desired config. Here we are creating interface looopback0 and assign an ipv4 address as well as description  
14.	Push the config to the device running config  
15.	Verify the operation was successful  

> snap

Login to the router to verify the change

> snap


We see from the logs the config was changed using NETCONF by user fisi and the interface loopback 0 comes up also we verify the configuration of interface loopback0 which has been created.

##### NETCONF Juniper demo
In this demo we write a sample code to get logical interfaces from a device  
First we launch the python interpreter. I’m using ipython  
1.	Import the manager module from the netconfclient (ncclient library) Your workstation is the manager whereas the network device is running the agent code  
2.	Import a library that will help us convert xml data returned by netconf to a python dictionary which is easier to navigate  
3.	Define the router and its login parameters  
4.	Connect to the router  
5.	Get running config from the device and convert the xml to a python dictionary at the same time  
6.	From the config use the keys to navigate to the data of interest which is the interface section in this case and print out the interface name and its logical interfaces names  
7.	Neatly close the session  

> snap

Important note is that since netconf uses structured data the commands are exactly the same across all vendors regardless of the difference in the cli syntax. Check out the other sample via https://github.com/hpreston/netconf_yang_blog


#### PyEZ
PyEZ is a python module that enables management and automation of devices running JunOS. PyEZ uses NETCONF to send remote procedure calls to JunOS devices with CLI support as well.  
Pre-requisite:  Netconf is enabled on the network device  

##### Junos PyEZ API demo  
In this demo we use PyEZ API to send commands to a junos device as well as equivalent CLI command  
1.	Import the Device module from junos library  
2.	Define login parameters  
3.	Open a session with the JunOS device  
4.	Make an rpc call to get bgp information from the device. The {‘format’:’text’} is used to convert the xml data returned by the device to a human readable format.  
5.	Convert the text data into a format as the one displayed when issuing a CLI command  
6.	Print the output  
7.	Get bgp information using cli command, the warning=False clause is to stop the warning message when using the cli method as the writers of PyEZ discourage using this method unless it is for debugging  
8.	Print the results  

> snmp

From the above demo we can verify we get the same output when using either method. The question now becomes how do we know the rpc equivalent commands since we obviously only know the CLI commands? Next demonstration shows how.


##### Getting the RPC command equivalent of the CLI command  
First im saving my username and password for the router as environment variables on my Ubuntu machine so that I can use the variable in the code and avoid typing the plain text password in the code then launch my python interpreter  
Procedures:  
1.	Import the Device module from junos library  
2.	Import the xml module for converting xml data  
3.	Import the os module that will be used to access the environment variables my username and password  
4.	Create a device object for the router  
5.	Open a session with the JunOS device  
6.	Get the rpc command equivalent for the cli command `show route`  
7.	Display the rpc command which is `get-route-information`  
8.	Get the rpc command equivalent for the cli command `show system alarms`  
9.	Display the rpc command which is `get-system-alarm-information`  
10.	Make an rpc call using the rpc command from step 9. Note `–` are replaced by `_`  
11.	Print the results from the device  
12.	Close the connection  

> snap

#### NAPALM
NAPALM stands for Network Automation and Programmability Abstraction Layer with Multivendor support is a Python library build by a community network engineers. It is a wrapper of different libraries that communicate directly with the network device. It also comes with a CLI tool you can use directly from your command prompt (or terminal) without running the python interpreter. The general idea behind this library is to create a standardized, multivendor interface for certain file and get operations

> snap

##### NAPALM with Cisco IOS
To enable NAPALM interaction with IOS devices configure 'ip scp server enable' on the device.

> snap

##### Demo
In the demonstration we use napalm to interact with a Cisco IOS device send configuration to the device and perform a rollback of the config. Note the rollback feature is not natively available on IOS devices but with the use of NAPALM we can easily achieve this.  
1.	Import napalm driver  
2.	Define login parameters  
3.	Create a napalm driver for the ios  
4.	Create a drive object and connect to the device  
5.	Use napalm to get facts about the device and we see from the list of interfaces we do not have loopback1 (We are goingto create loopback1 in the following steps)  
6.	Create configuration for loopback 1 interface following the ios CLI syntax  
7.	Load the configuration to the device (without saving the config)  
8.	Do a diff to compare the loaded config and the running config, the ‘+’ sign means we are adding the respective config. Note this feature is not natively available in regular Cisco IOS  
9.	Commit the configuration on the device  
10.	Get the facts from the device again, this time we see interface loopback1 is present since we added it with the commit operation on step 9  
11.	Issue a rollback operation to delete interface loopback1  
12.	Close the session  


> snap


##### Verification on the device  
The below snapshot shows what was happening in the background as we were configuring the device using napalm. I had opened the ssh session to the device and did a terminal monitor to see the log messages before I started operatin on NAPLM.  
Below is what happened to on the device  
-	First we see there was no interface loopback1  
-	Then on the log messages we see there was a config change by user fisi (from NAPALM) and the interface loopback1 comes up  
-	We can now see the configuration on interface loopback1 that was sent by napalm  
-	Then we see the rollback operation taking place as we issued the rollback command on NAPALM  
-	When we do a show run int lo1 it is not there anymore due to the rollback from NAPALM  

> snap

##### NAPALM with JunOS
1.	Import napalm driver  
2.	Define login parameters  
3.	Create a napalm driver for junos, create a driver object and connect to the device  
4.	Use napalm to get facts about the device  
5.	Create configuration for interface em2.555 following the junos CLI syntax  
6.	Load the configuration to the device (without saving the config) and print the diff between the active config and the loaded config   
7.	Commit the config  
8.	Use the CLI method to get configuration of interface em2.555  
9.	Print the config and see what we have added with the commit on step 7  
10.	Rollback the config  
11.	Use the CLI method to get configuration of interface em2.555  
12.	Print the config and we see there is none that’s because we rolled back the config on step 10  
13.	Close the session.  

> snap

#### NORNIR
Nornir is an automation framework written in python with multiple threading capabilities. Nornir has a simple inventory which uses two YAML files  
-	Hosts.yaml  
-	Groups.yaml (optional)  
See below snapshot of the files for my demo setup  

> snap

##### Demo
1.	Import the necessary libraries. In this demo we’ll use the netmiko_send_command in nornir to send commands to devices  
2.	Create a nornir object  
3.	Run the task using the netmiko_send_command function and specify the command  
4.	Use the nornir print function to print the results. The results are broken for router ‘big’ that is because the command we issued is not a valid junos command. The changed flag is false for both devices meaning nothing was changed on the device  

> snap

5.	Issue a command that works across both junos and ios  `show bgp summary`  
6.	Print the output we get clean output from both the routers big running junos and kh16 running ios. Again the change flag is false because nothing was changed in the config  

> snap

8.  Import a netmiko send config function for sending config to the device  
9.	Filter the hosts to only hosts running cisco_ios  
10.	Create config for interface loopback10  
11.	Send config to the device  
12.	Display the results and we see the change flag is now True, this is because we have actually changed the device config  
13.	Close the session.  

> snap


## Command line tools
### Napalm
NAPALM offers a command line utility that you can use directly from the command prompt for windows users or terminal for mac and linux users without using the python interpreter. Basically we can use all the get methods NAPALM provides right from the terminal such as getting facts, bgp neighbors and so much more.  

#### Demonstration
Using napalm cli to get facts about our cisco and junos devices  

> snap

### Ansible
Ansible uses playboooks to perform tasks. A play book consists of one or more tasks to be performed by modules. There are different modules for different operations such as backing up configuration, sending comands etc. These tasks are performed against specified host(s) as specified in the play  

In the below sample hosts file for ansible we have two groups of devices namely routers and vm with each group containing one device as displayed.

> snap

The playbook is written in yaml as displayed below

> snap

#### Running the playbook
First we confirm the contents of our directory by issuing a tree command and we see there is only the host file and junos_config_backup.yml file which is the playbook
Then we run the playbook specifying the host file with -i and it will play all the tasks defined in the play-book against the specified host, then we verify by using the tree command there is a new folder with the backup config for the router with ip 192.168.56.36

> snap

To learn more about ansible visit https://github.com/basondole/ansible

#### Playbook to configure junos device

The when statement in the tasks is a conditional. This means this task will only be done if the "configuring router interface via junos_config" task changes configuration so we need to use when conditional to check if the change flag is true from the first task. We run the playbook as shown below

> snap

On a separate terminal window we log in to the router and check the commit history, we can verify all the three tasks were executed on the device.

> snap

When we re-run the playbook again no changes are made, this is what is referred to as idempotency which essentially means only make changes when they are needed.

> snap

### Genie
Genie is both a library framework and a test harness that facilitates rapid development, encourage re-usable and simplify writing test automation. Genie was initially developed internally in Cisco, and is now available to the general public. Learn more about Genie at
https://developer.cisco.com/site/pyats/  
Genie uses a testbed file to define devices and their respective properties. Below is a sample testbed containing one device with a configured hostname KH16, the name of the device in the testbed must match exactly the configured hostname of the device in question. Note testbeds are written in yaml.

> snap

To check whether the testbed file is valid use the command pyats validate testbed <testbed-file-path> as shown below. If the testbed file is not valid the errors will be displayed, in this case the testbed file is valid however there is a warning message which we’ll ignore.
  
> snap

Genie offers a learn method which can be used to learn different things from the device(s) specified. In below sample we use learn to gather routing information from the device. 
 
The operation produces three files in the directory. The connection_KH16.txt file contains what genie did as it sets up the connection to the device. The routing_iosxe_KH16_console.txt contains all the commands sent to the device cli and their respective outputs and finally routing_iosxe_KH16_ops.txt contains the parsed structured data.

> snap

Refer below snapshot for the preview of the contents of each of the three files
 

### Talktous
Talk to us is a command line tool written by Paul S.I. Basondole on top of paramiko library. The tool provides a quick and easy way to send multiple commands to multiple devices and producing an intuitive output. These arguments can be loaded from a file or specified on cli at runtime. Talktous does not support ssh keys however it has the ability to cache ssh login username and password in an encrypted format and thus does not have to type in credentials every time an operation is ran on remote devices.
In below demonstration we use talktous to request for bgp summary information from the two devices 192.168.56.36 running junos and 192.168.56.63 running ios, we also see the login credentials were retrieved from a prior execution.


