---
title: Mikrotik RouterOS Initial Setup
date: 2024-01-28 13:45:22
categories: [Mikrotik,Router]
tags: [VLans,Networking,Routing]
---

RouterOS is a stand-alone operating system based on the Linux kernel. It powers MikroTik hardware devices but is also available for virtual machines.

For this example, we will start with a *clean install*. The system we will be using is the MikroTik hAP ax³ ([C53UiG+5HPaxD2HPaxD](https://mikrotik.com/product/hap_ax3)). Despite this, this configuration should be compatible with most Mikrotik RouterOS routers.

## Step 1 - Connect to the Router

Nowadays, Mikrotik ships its routers with a default configuration. Older devices come with a default configuration. Depending on the device, the port to access is `ether1`, `combo1` or `spf1`. By default, the IP is `192.168.88.1/24`. These older devices, you can access with username **admin** and **without password**. 

In some newer models, like the one we use, the default configuration comes with a login. You can find the credentials on the **sticker present on the device**.

## Step 2 - Reset Configuration

Before continuing, if you have a running configuration, it is recommended that you make a backup. You can find out how to do it [here](https://help.mikrotik.com/docs/display/ROS/Backup).

Some devices will not add a default IP to any port. Therefore, it is recommended that you have [WinBox](https://mikrotik.com/download) before proceeding. 

Resetting the system will give RouterOS its default settings. To accomplish this, use the following command:
```sh
/system reset-configuration no-defaults=yes skip-backup=yes
```
Output: 
```sh
Dangerous! Reset anyway? [y/N]:
```

Upon pressing `Y`, press `Enter`. The system will restart with the message: **System configuration will be reset.**

This action will result in a complete system reset. Therefore, the router should restart with its default configurations.

You can also reset it using the reset button present on the device. In most devices, you should **press the button and plug the power while pressing**. After 5 seconds, the LED should start blinking. Releasing the reset button at this time will reset the device.

Once the reset is complete, the router will be set to its factory settings. This means the wireless is not configured, and you must connect to it using a cable. 

## Step 3 - Creating a New User

Now that our device has the default settings. The first time we access it, there should be a prompt to add a new password. We recommend adding a strong password. The password set is for the admin account. 

We also recommend adding a new admin user and disabling the original admin. It is always a good policy to deactivate default accounts in a production environment.

To create a new user and remove the admin account, type the following commands adapting for your reality:

```sh
/user add name=<username> password=<password> group=full
/user remove admin
```

MikroTik has three default groups. We will leave a small description next if you want to create users with these groups. Remember, you can always have your groups.

| Group | Name | Policy |
| - | ------------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| 0 | name="read"  | policy=local,telnet,ssh,reboot,read,test,winbox,password,web,sniff,sensitive,api,romon,tikapp,!ftp,!write,!policy,!dude skin=default |
| 1 | name="write" | policy=local,telnet,ssh,reboot,read,write,test,winbox,password,web,sniff,sensitive,api,romon,tikapp,!ftp,!policy,!dude skin=default |
| 2 | name="full" | policy=local,telnet,ssh,ftp,reboot,read,write,policy,test,winbox,password,web,sniff,sensitive,api,romon,tikapp,!dude skin=default| 


## Step 4 - Configuring IP Access

With the freshly reset router, the next thing to do is to set an IP to a port so we can proceed with the configuration.

There are several ways to do this. For the time being, we will configure the IP for *ether2*:

```sh
ip address add address=192.168.88.1/24 interface=ether2
```
## Step 5 - Connect to the Internet

There are several ways to configure your router's access to the Internet. You can explore them on the [MikroTik website](https://help.mikrotik.com/docs/display/ROS/First+Time+Configuration).

In any case, we recommend adding a bridge interface for `WAN` connections and adding the *WAN port* to this bridge.

To create the Bridge interface, you need to run the following command:
```sh
interface bridge add name=WAN disabled=no
```

If you need to bypass the `WAN` connection, add another port to the WAN bridge, which should work.

### DHCP Client

This option will get an IP automatically. It is usually the fastest option when setting up a router under another router or when your ISP provides a DHCP Server. 

### ISP - MEO

In our case, we are using the *ISP from Portugal named MEO*. These steps may be different in your case. 

We will start by adding the `Internet vlan` for MEO on the `WAN` interface. In this case, MEO provides IP on **`VLan 12`**:

```sh
/interface bridge port add interface=ether5 pvid=12 bridge=WAN
/interface bridge set vlan-filtering=yes pvid=12 bridge=WAN
/interface bridge vlan add bridge=WAN vlan-id=12 tagged=ether5 untagged=WAN
```

After creating the VLan, all we need is to add the DHCP client:

```sh
ip dhcp-client add dhcp-options=hostname,clientid disabled=no interface=WAN
```



###	Verify the Connection

At this point in this guide, your router should already be able to connect to the Internet. Test this on the terminal by pinging `google.pt`.

## Step 6 VLANs

`VLans` (Virtual Local Area Networks) are virtual networks created within a physical network, allowing for segmentation and isolation of network traffic for improved security and network management.

For this example, we are adding 4 `VLans`:

| **VLAN NAME** | **ID** | **GateWay & Mask** | **Description**                             |
| ------------- | ------ | ------------------ | ------------------------------------------- |
| Users         | 10     | 172.16.10.1/24     | Home Users                                  |
| IoT           | 20     | 172.16.20.1/24     | IoT devices such as smart TV or Philips HUE |
| Guest         | 30     | 172.16.30.1/24     | Visitors Network                            |
| Servers       | 100    | 172.16.100.1/24    | Servers or Homelab                          |

### Creating VLANs

To make this work, we will start by adding the `VLans`:

```sh
/interface bridge add name="vlans"

/interface vlan
add interface="vlans" vlan-id=10 name="Users" disabled=no;

add interface="vlans" vlan-id=20 name="IoT" disabled=no;

add interface="vlans" vlan-id=30 name="Guest" disabled=no;

add interface="vlans" vlan-id=100 name="Servers" disabled=no;
```
Now that the `VLans` are created, we must give them their `IPs` and networks:

```sh
/ip address

add address=172.16.10.1/24 interface=Users;
add address=172.16.20.1/24 interface=IoT;
add address=172.16.30.1/24 interface=Guest;
add address=172.16.100.1/24 interface=Servers;
```

###	Configure DHCP for each VLan
Despite being ready to work, these VLans will not give an IP automatically. To provide IP, each VLan must have a `DHCP Server` configured. Here, we will explain how to configure these `DHCP Servers`.

#### Adding IP Pool

The first step is to create a `pool of IPs` for each `VLan`. A `pool of IPs` represents the range of `IPs` clients can have when connected to a `VLan`.

```sh
/ip pool

add name="Users-pool" ranges=172.16.10.15-172.16.10.254;
add name="IoT-pool" ranges=172.16.20.2-172.16.20.254;
add name="Guest-pool" ranges=172.16.30.2-172.16.30.254;
add name="Servers-pool" ranges=172.16.100.20-172.16.100.254;
```

#### Adding the Networks

To enable access to the Internet, each `VLan` must have a `gateway` and a **DNS** server to use. We can also define a domain name here, but that's out of the scope of this guide.


```sh

add address=172.16.10.0/24 dns-server=1.1.1.1,1.0.0.1,8.8.8.8,8.8.4.4 gateway=172.16.10.1 comment="User Network";

add address=172.16.20.0/24 dns-server=1.1.1.1,1.0.0.1,8.8.8.8,8.8.4.4 gateway=172.16.20.1 comment="IoT Network";

add address=172.16.30.0/24 dns-server=1.1.1.1,1.0.0.1,8.8.8.8,8.8.4.4 gateway=172.16.20.1 comment="Guest Network";

add address=172.16.100.0/24 dns-server=1.1.1.1,1.0.0.1,8.8.8.8,8.8.4.4 gateway=172.16.100.1 comment="Server Network";
```

#### Setup the DHCP Server for each network

The last thing to have the DHCP configuration set is to define the servers. 

```sh
/ip dhcp-server

add name="User-DHCP" interface=Users address-pool=Users-pool lease-time=1d add-arp=no disabled=no;

add name="IoT-DHCP" interface=IoT address-pool=IoT-pool lease-time=3d add-arp=no disabled=no;

add name="Guest-DHCP" interface=Guest address-pool=Guest-pool lease-time=30m add-arp=no disabled=no;

add name="Server-DHCP" interface=Server address-pool=Servers-pool lease-time=3d add-arp=no disabled=no;
```
> **Note:** The DHCP is working for the VLANs. You must associate them with a bridge or port to obtain IP on that port.
{: .prompt-info}

For example, we want to connect a computer on port `ether3` using `Users VLAN`. We will add this port to the `VLan Bridge` and enable `VLanfiltering` for this `bridge`.  

```sh
/interface bridge port add interface=ether3 pvid=10 bridge=VLAN-Bridge
/interface bridge set vlan-filtering=yes pvid=10 VLAN-Bridge
/interface bridge vlan add bridge=VLAN-Bridge vlan-id=10 tagged=VLAN-Bridge
/interface bridge vlan add bridge=VLAN-Bridge vlan-id=20 tagged=VLAN-Bridge
/interface bridge vlan add bridge=VLAN-Bridge vlan-id=30 tagged=VLAN-Bridge
/interface bridge vlan add bridge=VLAN-Bridge vlan-id=100 tagged=VLAN-Bridge
```

Now the port `ether3` should acquire `IP` from `Users VLAN`.

## Step 7 - Firewall

A firewall is a security barrier that monitors and controls network traffic, protecting systems from unauthorised access and potential threats.

Before setting up your firewall, we recommend you take a router backup. If, for some reason, a rule is misplaced, you may lose access to your router. 

Essentially, there are three chains on the firewall. There will be more if you create custom rules or custom chains. But for now, let us focus on the basic ones:

| **Chain**   | **Description**                                                    |
| ------- | -------------------------------------------------------------- |
| input   | Refers to packets/connections that end in the router.          |
| output  | Refers to packets/connections from the router.                 |
| forward | Refers to packets/connections just passing through the router. |

###	Protect the Router

We recommend creating a list of networks which may access the router. Typically, this is the management `VLan` In our case, we will give this access to the `Users VLan`:

```sh
/ip firewall address-list add list=allowed-to-router address 172.16.10.0/24
```

Then, we can build our firewall to protect the router:
```sh
/ip firewall filter

add chain=input action=accept src-address-list=allowed-to-router in-interface=!WAN

add chain=input connection-state=established,related action=accept comment="Accept established or related connections";

add chain=input connection-state=invalid action=drop comment="Drop invalid connections";

add chain=input src-address-list=!allowed-to-router protocol=icmp action=drop comment="Prevent ICMP from outside";

add chain=input in-interface=WAN action=drop comment="Block everything else from outside";

```
These rules will prevent access from the Internet to the router and also prevent pings to it. If your router doesn't answer, bots won't know it is there.

### Protect the Users

```sh
/ip firewall filter

add chain=forward action=fasttrack-connection connection-state=established,related comment="Fast-track for established or related connections";

add chain=forward action=accept connection-state=established,related comment="Accept established or related connections";

add chain=forward action=drop connection-state=invalid;

add chain=forward action=drop connection-state=new connection-nat-state=!dstnat in-interface=WAN comment="Drop access to clients behind NAT from WAN"
```
Filter the `IoT` and `Guest VLans`:

```sh
add action=accept chain=forward comment="Allow IoT to the Internet" connection-state=new in-interface=IoT out-interface=WAN
add action=accept chain=forward comment="Allow VPN to Internet" in-interface=loopback out-interface=WAN
add action=accept chain=forward comment="Allow Users to the Internet" connection-state=new in-interface=Users out-interface=WAN
add action=accept chain=forward comment="Allow Servers to the Internet" connection-state=new in-interface=Servers out-interface=WAN
add action=accept chain=forward comment="Allow Users to Servers" in-interface=Users out-interface=Servers

```

There are still some holes in the firewall. To cover these, we recommend adding two last rules:
```sh
add action=drop chain=forward comment="Prevent unwanted access"
add chain=input action drop
```


To configure the `DNS` servers, just run the following command:
```sh
/ip dns set servers=1.1.1.1,1.0.0.1,8.8.8.8,8.8.4.4
```

###	Allowing VLANs to the Internet

The `DHCP server` is up, the `VLANs` are up, and the `FIrewall` is ready. Still, users need access to the Internet. We must masquerade their connections and send them to the `WAN` port. Then, users will be ready to speak to the world.

```sh
/ip firewall nat
  add chain=srcnat out-interface=Internet action=masquerade comment="Access to the internet"
```

## Step 8 - Wireless

The last part of this guide will help set up this router's wireless. Since, in this case, the router will work alone, we will not use `Capsman`. Instead, we will configure the wireless in a stand-alone mode.

We start by creating the `datapath` for each `VLAN` that will use the wireless:

```sh
/interface wifiwave2 datapath
add name="Users-Datapath" bridge=VLAN-Bridge vlan-id=10 disabled=no comment="Users Datapath";
add name="IoT-Datapath" bridge=VLAN-Bridge vlan-id=20 disabled=no comment="IoT Datapath";
add name="Guest-Datapath" bridge=VLAN-Bridge vlan-id=30 disabled=no comment="Guest Datapath";
```

Now, we will create the passwords for each network:

```sh
/interface wifiwave2 security
add name=User-Sec authentication-types=wpa2-psk,wpa3-psk encryption=ccmp,gcmp,ccmp-256,gcmp-256 passphrase=<password> comment="Users";
add name=IoT-Sec authentication-types=wpa2-psk,wpa3-psk encryption=ccmp,gcmp,ccmp-256,gcmp-256 passphrase=<password> comment="IoT";
add name=Guest-Sec authentication-types=wpa2-psk,wpa3-psk encryption=ccmp,gcmp,ccmp-256,gcmp-256 passphrase=<password> comment="Guest";
```

Select the channels for our configuration:
```sh
/interface wifiwave2 channel
add name=ch-2ghz frequency=2412,2432,2472 width=20mhz
add name=ch-5ghz frequency=5180,5260,5480,5500 width=20/40/80mhz
```

To warp all, create  the configuration and link everything to it:

```sh
/interface wifiwave2 configuration
add name=User-conf-2ghz ssid="<name of user wifi>" contry="<your country>" mode=ap security=User-sec datapath=Users-Datapath channel=ch-2ghz
add name=IoT-conf-2ghz ssid="<name of user wifi>" contry="<your country>" mode=ap security=IoT-sec datapath=IoT-Datapath channel=ch-2ghz
add name=Guest-conf-2ghz ssid="<name of user wifi>" contry="<your country>" mode=ap security=Guest-sec datapath=Guest-Datapath channel=ch-2ghz
add name=User-conf-5ghz ssid="<name of user wifi>" contry="<your country>" mode=ap security=User-sec datapath=Users-Datapath channel=ch-5ghz
add name=IoT-conf-5ghz ssid="<name of user wifi>" contry="<your country>" mode=ap security=IoT-sec datapath=IoT-Datapath channel=ch-5ghz
add name=Guest-conf-5ghz ssid="<name of user wifi>" contry="<your country>" mode=ap security=Guest-sec datapath=Guest-Datapath channel=ch-5ghz
```
We are ready to start the wifi:

```sh
/interface wifiwave2
set wifi1 configuration=User-conf-5ghz name=wifi-5-User disabled=no
add name=wifi-5-IoT configuration=IoT-conf-5ghz master-interface=wifi-5-User disabled=no
add name=wifi-5-Guest configuration=Guest-conf-5ghz master-interface=wifi-5-User disabled=no
set wifi2 configuration=User-conf-2ghz name=wifi-2-User disabled=no
add name=wifi-2-IoT configuration=IoT-conf-2ghz master-interface=wifi-2-User disabled=no
add name=wifi-2-Guest configuration=Guest-conf-2ghz master-interface=wifi-2-User disabled=no
```

## Conclusion

This tutorial has guided you through the comprehensive setup and configuration of a MikroTik router running RouterOS. Starting from a clean install, we've covered essential steps, including connecting to the router, resetting its configuration, creating new users, and setting up IP access. We've also delved into configuring VLANs, establishing a firewall, and enabling internet access. Finally, we've set up wireless connectivity, ensuring your network is segmented and secure.

By following these steps, you should have a robust, secure, and well-organised network ready to handle the demands of various devices and users. Remember, each network environment is unique, so feel free to adapt the configurations to suit your specific needs better. MikroTik's extensive documentation and community forums are valuable resources if you encounter any issues or need further customisation.

Happy networking! 🚀🌐