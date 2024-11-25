# CISCO 3550 Series Switch Configuration
CISCO 3550 Series Switch Configuration

official doc: https://www.cisco.com/en/US/docs/switches/lan/catalyst4000/7.5/configuration/guide/cli_support_TSD_Island_of_Content_Chapter.html

24 FastEthernet ports 0/x , 2 GigabitEthernet ports 0/x

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
