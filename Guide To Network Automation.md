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

<telnet images>

#### Paramiko
Paramiko uses the SSH protocol to communicate with devices and sends raw cli commands as it will be shown in the demonstration. In very light language paramiko does what the telnetlib does but with SSH instead of telnet however paramiko is not a standard python library and must be installed separately.  
To install paramiko on machine with python and pip installed do: `pip install paramiko`  
Pre-requisite: 
1.	ssh is enabled on the remote device  
2.	paramiko library is installed on the local working station  

##### Paramiko demo
Again, the syntax for each of the below actions is shown in the next snapshot
