# CISCO 3550 Series Switch | CISCO 2900 Series Router Configuration
CISCO 3550 Series Switch Configuration
[Basics on youtube](https://www.youtube.com/watch?v=jjbJbbFYAkY)

official doc: https://www.cisco.com/en/US/docs/switches/lan/catalyst4000/7.5/configuration/guide/cli_support_TSD_Island_of_Content_Chapter.html

CISCO 3550 Series Switch: 24 FastEthernet ports 0/x , 2 GigabitEthernet ports 0/x

CISCO 2900 Series Router: 2 GigabitEthernet ports 0/x

During First Boot:
* First, it will initialize the flash memory[memory that retains data in the absence of power supply ] (because it contains IOS image of the Switch )
* Load the IOS image from the flash memory (The IOS image is compressed so the switch uncompresses the image and loads it in RAM)
* IOS is now up and running, it also initializes the flash memory
* IOS starts with a POST (Power on Self Test) for some of the switch components (registers, memory,loopback,power controller,CAM subsystem,port loopback)


Modes: user mode ---> privilege mode ---> config mode

		      >enable  	   #configure terminal (Global Configuration mode)
        
1. Connect to switch console (serial communication)

   * Open a putty/Teraterm session (The connected COM PORT (check in device manager),select Serial)

2. Configurations are done in config mode; Show commands executed in privilege mode.

3. Exit : exit / CTRL+Z

4. Change hostname
   * --->go to configuration mode
   * hostname cisco_switch
  
5. Change privilege (enable) mode password
    * --->go to configuration mode
    * enable password cisco
    
6. Check IP interfaces
    * --->go to privilege mode
    * show ip interface brief
    
7. Configuring IP to a port
    * --->go to configuration mode [Switch(config)]
    * interface FastEthernet0/23 [Switch(config-if)]
    * no switchport : changes the port from L2 to L3 interface
    * ip address 192.168.20.100 255.255.255.0 : assign ip address 
    * no shutdown : change the state of an interface from 'shutdown' (disabled) to active (enabled).
    * --->go to privilege mode ; write memory :save configuration changes to NVRAM and make them permanent. (after reboot the setup method turns from manual to NVRAM)
    * Now connect the FastEthernet0/23 with an ethernet cable.
      
8. Test Ping
    * Assign IP to your ethernet connected interface in the pc . (Go to network settings>go to ethernet settings>Assign IP address 192.168.20.101 255.255.255.0 such that the switch interface & pc interface in same subnet(Thats what switches are for!!!))
    * Open CMD on pc. ping 192.168.20.100 (if fails: turn off the firewall configuration). Similarly ping 192.168.20.101 from the switch serial connection.
    * To access the switch via ssh. Need to create a username and password where default will be cisco & cisco. 
      * --->go to configuration mode (https://community.cisco.com/t5/switching/ssh-access-denied-while-accessing-router/td-p/3013811)
      * username test password test
      * line console 0 : (sets the configuration to the line configuration : the console the switch is plugged in)
      * no aaa new-model :  authentication, authorization, and accounting is removed. The login method is switched back to line authentication
      * line vty 0 15 : configures the first sixteen virtual terminal lines, numbered from 0 to 15 (support 16 simultaneous remote connections)
      * transport input ssh : allow TCP/IP SSH protocol only to access the virtual terminal lines. (https://study-ccnp.com/transport-input-ssh-telnet-all-none-keywords/)
      * login local :tell the switch to refer to a local database of usernames and passwords for authentication
9. Configuring vlan
    [Inter-VLAN Routing](https://www.ciscopress.com/articles/article.asp?p=3089357&seqNum=6) (possible with Layer 3 Switch)
   ![image](https://github.com/user-attachments/assets/632ddd93-300c-4254-b162-47e01e741435)

   *  Check vlan
		* (in privilege mode)  show vlan
   * Create two vlans
		* --->go to configuration mode
		* vlan 10
		* name groupA
		* vlan 20
		* name groupB
		* exit
   * Configure IP address to the vlans (will serve as the default gateway to the hosts in the respective vlans)
		* --->go to configuration mode
		* interface vlan 10
		* ip address 192.168.3.100 255.255.255.0
		* no shutdown
		* interface vlan 20
		* ip address 192.168.4.100 255.255.255.0
		* no shutdown
		* exit
    * Configure access ports connecting to the hosts & assign them to respective vlans
		* --->go to configuration mode
		* interface range FastEthernet0/1-3
		* switchport mode access (command forces the port to be an access port while and any device plugged into this port will only be able to communicate with other devices that are in the same VLAN)
		* switchport access vlan 10
		* interface range FastEthernet0/20-22
		* switchport mode access
		* switchport access vlan 20
		* exit
     * Test Ping
		* PC1→vlan 10 **SUCCESS**
		* PC2→vlan 20 **SUCCESS**
		* PC1→PC2 vice versa **FAILED** (No inter vlan routing configured)
		* PC1→vlan 20 (PC2→vlan 10) **FAILED** (No inter vlan routing configured)
      * Enable inter-vlan IP routing (**SUCCESS** after disabling wifi)
		* --->go to configuration mode
		* ip routing
		* show ip route (in privilege mode)
10. Router On a stick (VLAN Routing with external Router)
[VLAN Routing using external Router](https://networklessons.com/switching/intervlan-routing)
![image](https://github.com/user-attachments/assets/e7b20703-4f12-4a82-b29c-c6ea5a0df4eb)

	* On SWITCH:
		* Configure vlans, PCs following previous steps.
		* Disable ip routing in the switch
			* --->go to configuration mode
			* no ip routing
			* exit
		* Create 802.1Q trunk between the switch and the router 
			* --->go to configuration mode
			* interface FastEthernet 0/12
			* switchport trunk encapsulation dot1q (common trunking protocol : 802.1Q inserts a VLAN tag on the frames)
			[802.1Q encapsulation](https://networklessons.com/switching/802-1q-encapsulation-explained)
			* switchport mode trunk
			* switchport trunk allowed vlan 10,20 (allow vlan 10,20 only via trunk port)
	* On ROUTER:
		* Create 2 sub-interfaces & assign the vlans
			* --->go to configuration mode
			* interface fa0/0.10
			* encapsulation dot1Q 10
			* ip address 192.168.3.33 255.255.255.0
			* exit
			* interface fa0/0.20
			* encapsulation dot1Q 20
			* ip address 192.168.4.44 255.255.255.0
			* exit
   	* Check Routing
		* show ip route
	* Set Default Gateway of the PCs to the ip addresses of vlan subinterfaces at the Router (routing in SWITCH is disabled)
		* PC1 :192.168.3.33
		* PC2 :192.168.4.44
	* Test Ping
		* PC1→PC2 vice versa **SUCCESS** 
		* PC1→ default gateway **SUCCESS** 
		* PC2 → default gateway **SUCCESS**
11. Static Routing
    [Static Routing in Cisco 2 router connections](https://www.geeksforgeeks.org/implementation-of-static-routing-in-cisco-2-router-connections/)
    
    ![image](https://github.com/user-attachments/assets/88143c71-2c9f-45ce-b3b0-f0bc08128bd5)
    * On ROUTER:
		* Configure interface ip addresses as in previous tasks
		* Assign Static Route to particular router
			* --->go to configuration mode
			* ip route 192.168.5.0 255.255.255.0 192.168.10.11
			* exit
    * On SWITCH:
		* Configure interface ip addresses as in previous tasks
		* Assign Static Route to particular L3 switch 
			* ---> go to configuration mode
			* ip route 192.168.3.0/24 192.168.10.10
			* exit
    * Set default gateways
		* PC1: 192.168.3.100
		* PC2:192.168.5.10
    * Test Ping
		* PC1→PC2 vice versa **SUCCESS**
    * Add route from PCs (disable routing in router & switch)
		* PC2:route add 192.168.3.0/24 192.168.5.10 
		* PC1:route add 192.168.5.0/24 192.168.3.100
     * Test Ping 
		* PC1→PC2 vice versa **SUCCESS**
       
12.Dynamic Routing (**R**oute**I**nformation**P**rotocol)

[Dynamic Routing in Cisco for connecting two routers](https://www.geeksforgeeks.org/implementation-of-rip-routing-in-cisco-for-connecting-two-routers/)

[Configure RIP protocol](https://networklessons.com/rip/how-to-configure-rip-on-a-cisco-router)

* On ROUTER:
	* Configure interface ip addresses as in previous tasks
	* Configure rip
		* --->go to configuration mode
		* router rip
		* network 192.168.3.0
		* network 192.168.10.0
		* exit
* On SWITCH:
	* Configure interface ip addresses as in previous tasks
	* Configure rip
		* ---> go to configuration mode
		* router rip
		* network 192.168.5.0
		* network 192.168.10.0
		* exit
* Set default gateways
	* PC1: 192.168.3.100
	* PC2:192.168.5.10
* Test Ping
	* PC1→PC2 vice versa **SUCCESS**
* Check rip database
	* show ip rip database

13. OSPF Configuration

    [OSPF Configuration](https://study-ccna.com/ospf-configuration/)
    
    [Basic OSPF Configuration](https://networklessons.com/ospf/basic-ospf-configuration)

 ![image](https://github.com/user-attachments/assets/f25b073d-d93e-4cb6-8aed-89704ed04c6a)

* On R1:
	* Configure interface ip addresses & PC ip addresses as in previous tasks
	* Configure OSPF
		* --->go to configuration mode
		* router ospf 1 (number “1” is process id for the ospf instance)
		  [OSPF process id](https://netseccloud.com/understanding-ospf-process-id-what-it-is-and-how-it-works#:~:text=The%20OSPF%20process%20ID%20might,distinguish%20between%20different%20OSPF%20instances.)
		* network 192.168.3.0 0.0.0.255 area 0
		* network 192.168.10.0 0.0.0.255 area 0 
		(0.0.0.255 => a wildcard mask(reverse subnet mask)) [wildcard masking](https://www.pynetlabs.com/what-is-wildcard-mask-in-networking/)

		indicate which bits of an IP address must match and which bits do not matter.**(0 means that the equivalent bit must match,1 means that the equivalent bit does not matter)**
   
		OSPF is a dynamic routing protocol that mainly makes use of link-state information to calculate the best routes for IP packets within a network.
		[Why area is needed in ospf?](https://networkengineering.stackexchange.com/questions/4997/what-is-the-advantage-of-an-area-in-ospf-configuration)
		* exit

* On R2:
	* Configure interface ip addresses & PC ip addresses as in previous tasks
	* Configure OSPF
		* --->go to configuration mode
		* router ospf 5 
		* network 192.168.5.0 0.0.0.255 area 0
		* network 192.168.10.0 0.0.0.255 area 0 
		* exit

* Check OSPF 
	* show ip ospf neighbor
	* show ip route (OSPF routes are indicated by character “O”)
* Set default gateways
	* PC1: 192.168.3.100
	* PC2:192.168.5.100
* Test Ping
	* PC1→PC2 vice versa **SUCCESS**
14. DHCP server & Client

   [Configure Cisco Router as DHCP Server ](https://study-ccna.com/configure-cisco-router-as-dhcp-server/)
  
   [Configure DHCP Client on Cisco IOS](https://networklessons.com/cisco/ccie-routing-switching/dhcp-client-on-cisco-ios)

   ![image](https://github.com/user-attachments/assets/66c23ea7-7170-41fa-87b1-6f98a9a450cd)

* On R2:
	* Configure DHCP Server
		* --->go to configuration mode
		* ip dhcp excluded-address 192.168.1.1 192.168.1.10 (Exclude IP addresses from being assigned by DHCP. (Exclude IP addresses that need not to be changed dynamically: Default gateway, servers)
		* ip dhcp pool MY_POOL (Create a dhcp pool)
		* network 192.168.1.0 255.255.255.0 (Define a subnet that will be used to assign IP addresses to hosts with the network)
		* default-router 192.168.1.1 (Set the default gateway of the network)
		* dns-server 192.168.1.2 (not necessarily) 
		* exit
	* Check DHCP 
		* show ip dhcp binding (shows leased Addresses by the dhcp clients)
		* show ip dhcp pool (display information about configured pools)
* On R1:
	* Configure DHCP Client
		* --->go to configuration mode
		* interface fa0/0
		* ip address dhcp
		* no shutdown
		* end
	* Check ip assignment method
		* show ip interface brief

15. Dynamic ARP inspection

    [Dynamic ARP Inspection ](https://networklessons.com/switching/dai-dynamic-arp-inspection)

    ![image](https://github.com/user-attachments/assets/5ec95a17-f7da-4649-90d5-295fe8907835)

 * On SWITCH:
	* Configure a vlan comprising of all three interfaces of the switch as “vlan123”.
	* Configure DHCP Snooping
		* --->go to configuration mode
		* ip dhcp snooping
		* ip dhcp snooping vlan 123
		* exit
		* --->go to configuration mode
		* interface fa0/3
		* ip dhcp snooping trust (Make the interface that connects to the DHCP server trusted )
		* end
* On DHCP SERVER
	* Configure DHCP server as in the previous task as necessary.(iii,iv)
* On HOST:
	* Configure host interface as a DHCP client as in the previous task.
* On SWITCH:
	* Check DHCP binding as in the previous task (Ip address assigned to the Host can be seen)
	* Enable Arp Inspection for vlan123
		* --->go to configuration mode
		* ip arp inspection vlan 123
		* exit
* On ATTACKER:
	* Configure Ip address of Attacker same as of Host (192.168.1.1 255.255.255.0)
	* PING from ATTACKER to DHCP ROUTER:
		* ping 192.168.1.254 **FAILED** (check on switch {show ip arp inspection}: all ARP requests from the attacker are dropped)
        



    

