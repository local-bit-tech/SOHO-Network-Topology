# VLAN's and STP

## VLAN name Configuration
First, We will configure VLAN's on Switch 1 and Switch 2.
VLAN's are important to separate your broadcast domains to make your network topology safer.

We will go into privileged exec mode then into global configuration mode to configure the switches.

To configure the VLAN's and their names we run these commands:
```text
SW1(config)# vlan 10
SW1(config-vlan)#name HR
SW1(config-vlan)#vlan 20
SW1(config-vlan)#name IT
````

For both switches, we make sure VLAN's 10 and 20 are created with names HR and IT, respectively.


## VLAN Switchport Access
We will now place the respective PCs on their VLAN's for their departments.
We also need to configure them as access ports since these PCs will only be sending traffic from one VLAN

```text
SW1(config)#int f0/3
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 10
SW1(config-if)#int f0/4
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 20
```
The configurations on SW2 will also be the same since the same ports are being connecting over there also. 

<img width="697" height="200" alt="image" src="https://github.com/user-attachments/assets/e8a38d8f-fff6-4fb0-9397-2e7201fd764e" />

This image shows vlan 10 (HR) has the port F0/3 and vlan 20 (IT) has the port F0/4.



## STP Root Configurations
We now will configure which switch will be the primary and secondary roots for each vlan.
This is important to load balance and have redundancy between switches.

Configurations on SW1 for this:
```text
SW1(config)#spanning-tree vlan 10 root primary
SW1(config)#spanning-tree vlan 20 root secondary
```

This would be the same configurations on SW2 except the roots get reversed.

<img width="758" height="410" alt="image" src="https://github.com/user-attachments/assets/12339875-fcad-4ee4-846c-d4991d171d4c" />

From the image, we see that SW1 is the root bridge for vlan 10 and SW2 is the root primary for vlan 20.
## Portfast

Portfast is important for STP because it allows end devices (like PCs) to immediately start forwarding traffic
since they would not have to go through a listening and learning state.


Configuring Portfast for both switches
```text
SW1(config)#int range f0/3-4
SW1(config)#spanning-tree portfast
```

Since both switches are connected to end devices on their respective fastethernet interfaces 3 and 4,
we can use the same commands on both devices.

__NOTE: If we plan on connecting a switch in place of a PC with portfast enabled, we must turn off portfast or put on bpduguard__

## Trunk Links

We have to establish a trunk link between SW1 and SW2. These trunk links allow for the switches to communicate multiple vlan's on a singular port.
We will make sure traffic for vlan's 10 and 20 are passed in this trunk link.

Configuring trunk links
```text
SW1(config)#int range f0/1-2
SW1(config)#switchport mode trunk
SW1(config)#switchport trunk allowed vlan 10,20
```
These would be the same configurations for SW2. We should not allow for all vlans to communicate in this trunk as this would cause security concerns.

<img width="588" height="218" alt="image" src="https://github.com/user-attachments/assets/9426329a-f01c-47ed-b7ea-0281834bfb2f" />

From the image, we can see that the trunk between f0/1 and f0/2 are only allowing vlans 10 and 20 to pass through.


