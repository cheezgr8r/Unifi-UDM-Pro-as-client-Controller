# How to use a UDM Pro as a client-controller

The purpose of this is to setup a UDM Pro on a network where there is already a firewall & DHCP lease manager.  The environment this was developed, tested, and deployed on uses PFsense for firewall and DHCP management, and a UCKG2 for network appliance management and a UNVR for security appliance management.  I will try to outline why I made certain choices, most were to segregate controllers and appliances to prevent possible infighting.

## Some notes before starting
1. The UDM Pro is not happy without a WAN connection, to satisfy this we will end up plugging the WAN port directly into a UDM Pro switch port.
2. This setup was tested using a TP-Link managed switch, and deployed using a USW Pro, separate sections will be outlined below for each case.
3. The LAN and WAN can be on different VLAN's or the same VLAN.  Because of the environment this is being used on, the WAN is on a management VLAN and the LAN is on the Access controller VLAN.
4. The primary goal of this was to deploy the Unifi Access system while ensuring stability and security without having to rebuild an existing infrastructure.  If it seems like it's overkill, it probably is, but it is my belief that security and stability cannot be guaranteed without overkill. 
5. This was only tested on the Access controller of the UDM Pro, it is probable that this same setup can be used to deploy other controllers of the UDM Pro in a similar fashion, but they have not been tested and I cannot speak to the method on deploying them.  If you are starting from scratch it may make more sense to deploy the Network controller, the only reason this avenue was not explored was the view that this setup would be temporary and migrating from the UCKG2 would be extra effort.
6. In resiliency testing (power cycling) it was observed that the Management network for access devices shifted from the Access VLAN to the default LAN.  This was observed one time, and may have been user error (I set it to LAN without realizing it), but because of this the Access VLAN was deleted and the LAN was placed on the Access VLAN.


## Initial setup
1. Identify desired setup and CIDR blocks to be used, this is also a good time to create the Access VLAN if that will be used.  Decide if LAN and WAN will be on the same VLAN and if that VLAN will be used for managing access devices.
2. Connect the UDM Pro WAN port to the network, and the terminal to be used for setup to the UDM Pro.
3. Power on the UDM Pro and go through the setup process.

## Configure UDM Pro with WAN on Management VLAN and LAN on Access VLAN
1. Create a static mapping for the WAN port in your DHCP manager, this is optional but something I like to do.
2. Under *Network > Settings > Routing & Firewall > Firewall > Rules IPv4 > LAN IN*:
   1. Click *+ CREATE NEW RULE*
   2. Enter **Name**
   3. Under **Action** click **Accept**
   4. Click **Save**
3. Repeat *Step 2* for *LAN OUT* and *LAN LOCAL*.
4. Under *Network > Settings > Controller*:
   1. Set the **Controller Hostname/IP** to an IP address on the Access VLAN (ex. 10.1.1.20).
***This IP address will not show up in the lease manager, make sure it is out of the pool of leases and not mapped or static to another device***
   2. Click **Save**  
5. Under *Network > Settings > Networks > LAN*:
   1. Set the **Gateway IP/Subnet** to the same IP address in *Step 4* followed by `/24` (ex. 10.1.1.20/24)
   2. Set the **DHCP Mode** to *None*
   3. Click **Save**
6. Under *Network > Settings > Networks > CREATE NEW NETWORK*
   1. Enter **Name** of Mangement VLAN
   2. Select **VLAN Only**
   3. Enter VLAN Tag under **VLAN*
   4. Click **Save**
7. Repeat *Step 6* for Access VLAN
8. Under *Network > Settings > Networks > Profiles > Switch Ports > + ADD NEW PORT PROFILE*:
   1. Enter **Profile Name**
   2. Under **Native Network** select Management VLAN from *Step 6*
   3. Under **Tagged Networks** select **Select all**
   4. Click **Save**
9. Under *Network > Devices > UniFi Dream Machine Pro > Ports*:
   1. Select a Port the WAN will plug into (Port 8 is directly next to the WAN port and is perfect for a short patch panel cable) and edit that port.
   2. Under **Switch Port Profile** select the profile made in *Step 8*.
   3. Click **Save**
10. Turn off the UDM Pro, connect the WAN port and the Port used in *Step 9*.
11. On the separate Unifi Network Controller create the Access VLAN network as in *Step 6*.
12. On the separate Unifi Network Controller create a Switch Port Profile for the UDM Pro. Under *Network > Settings > Networks > Profiles > Switch Ports > + ADD NEW PORT PROFILE*:
   1. Enter **Profile Name**
   2. Under **Native Network** select Access VLAN from *Step 11*
   3. Under **Tagged Networks** select the Management VLAN
   4. Click **Save**
13. On the Unifi Switch, select a port to plug the UDM Pro into and set the *Switch Port Profile* to the profile created in *Step 12*. 
14. Plug in the UDM Pro in to the Port from *Step 13* (The default profile is *ALL* so any convenient port on the UDM Pro will work) and power on the UDM Pro.
15. On the UDM Pro the WAN address should show a management VLAN address, if a static mapping was made in *Step 1* the UDM Pro should grab that address.  The LAN Address should match the address from *Step 4*.  You should be able to access the UDM Pro from the Management and Access VLANs at their respective addresses, and the UDM Pro should also be accessible when you log into [unifi.ui.com](https://unifi.ui.com)

## Configure UDM Pro with WAN and LAN on same VLAN
*This section assumes the VLAN is already created and in use*
1. Create a static mapping for the WAN port in your DHCP manager, this is optional but something I like to do.
2. Under *Network > Settings > Routing & Firewall > Firewall > Rules IPv4 > LAN IN*:
   1. Click *+ CREATE NEW RULE*
   2. Enter **Name**
   3. Under **Action** click **Accept**
   4. Click **Save**
3. Repeat *Step 2* for *LAN OUT* and *LAN LOCAL*.
4. Under *Network > Settings > Controller*:
   1. Set the **Controller Hostname/IP** to an IP address on the Access VLAN (ex. 10.1.1.20).
***This IP address will not show up in the lease manager, make sure it is out of the pool of leases and not mapped or static to another device, it can also be the same address from Step 1***
   2. Click **Save**  
5. Under *Network > Settings > Networks > LAN*:
   1. Set the **Gateway IP/Subnet** to the same IP address in *Step 4* followed by `/24` (ex. 10.1.1.20/24)
   2. Set the **DHCP Mode** to *None*
   3. Click **Save**
6. Under *Network > Settings > Networks > CREATE NEW NETWORK*
   1. Enter **Name** of Access VLAN
   2. Select **VLAN Only**
   3. Enter VLAN Tag under **VLAN*
   4. Click **Save**
7. Under *Network > Settings > Networks > Profiles > Switch Ports > + ADD NEW PORT PROFILE*:
   1. Enter **Profile Name**
   2. Under **Native Network** select **None**
   3. Under **Tagged Networks** select **Select all**
   4. Click **Save**
8. Under *Network > Devices > UniFi Dream Machine Pro > Ports*:
   1. Select a Port the WAN will plug into (Port 8 is directly next to the WAN port and is perfect for a short patch panel cable) and edit that port.
   2. Under **Switch Port Profile** select the profile made in *Step 7*.
   3. Click **Save**
9. Turn off the UDM Pro, connect the WAN port and the Port used in *Step 8*.
10. On the Unifi Switch, select a port to plug the UDM Pro into and set the *Switch Port Profile* to the Access VLAN, or switch port profile with the Access VLAN as the Native VLAN.
11. Plug in the UDM Pro in to the Port from *Step 10* (The default profile is *ALL* so any convenient port on the UDM Pro will work) and power on the UDM Pro.
12. On the UDM Pro the WAN address should show a management VLAN address, if a static mapping was made in *Step 1* the UDM Pro should grab that address.  The LAN Address should match the address from *Step 4*.  You should be able to access the UDM Pro from the Management and Access VLANs at their respective addresses, and the UDM Pro should also be accessible when you log into [unifi.ui.com](https://unifi.ui.com)

## Adding a Unifi Access Hub to the network.
1. If using cascaded switches, ensure interfacing switch port profiles have the Access VLAN tagged.
2. On the port the Unifi Access Hub will be plugged into change the **Switch Port Profile** to **Access VLAN**.

## If you want to use a separate Access VLAN from the LAN that is not managed by the UDM Pro
*You may want to do this if the LAN and WAN are on a management VLAN, after figuring out how to do this I decided I didn't like the configuration, but thought I would pass along the steps.
1. Under *Network > Settings > Networks > + Create New Network*:
   1. Enter **Name**
   2. Enter VLAN tag under **VLAN**
   3. Enter an unused address on the Access VLAN outside of the pool followed by `/24` (ex 10.1.1.10/24)
   4. Enter **Domain Name** (ex. localdomain)
   5. Under **DHCP Mode** click **None**
   6. Click **Save**
2. The Access VLAN will now be available to select in the Access controllers Preferences page but will be managed by an appliance other than the UDM Pro.
3. Update switch port profiles to tag the Access VLAN so traffic will be passed over the network

*If you are using a managed switch not Unifi: The port the UDM Pro is plugged into should have the VLAN the WAN port is on should be the PVID VLAN and the untagged VLAN with the Access VLAN (if applicable) tagged.

** Closing Notes
1. The Access Hub will pass traffic if a computer is plugged in, allowing a malicious actor hardwire access to your LAN through the UA-Lite or UA-Pro.  I addressed this using MAC address control on the Access VLAN.
2. The UA-Pro allows far too much admin control at a remote location for my comfort, and logging into the UA-Pro to adjust volume is obnoxious.  I sincerely hope Ubiquiti tightens the security on the UA-Pro.
