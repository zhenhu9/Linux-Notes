### Chapter 44. Using and configuring firewalld

A firewall is a way to protect machines from any unwanted traffic from outside. It enables users to control incoming network traffic on host machines by defining a set of firewall rules. These rules are used to sort the incoming traffic and either block it or allow through.

Note that firewalld with nftables backend does not support passing custom nftables rules to firewalld, using the --direct option. 

#### When to use firewalld, nftables, or iptables

The following is a brief overview in which scenario you should use one of the following utilities:

- firewalld: Use the firewalld utility for simple firewall use cases. The utility is easy to use and covers the typical use cases for these scenarios.
- nftables: Use the nftables utility to set up complex and performance critical firewalls, such as for a whole network.
- iptables: The iptables utility on Red Hat Enterprise Linux 8 uses the nf_tables kernel API instead of the legacy back end. The nf_tables API provides backward compatibility so that scripts that use iptables commands still work on Red Hat Enterprise Linux 8. For new firewall scripts, Red Hat recommends to use nftables. 

Important

To avoid that the different firewall services influence each other, run only one of them on a RHEL host, and disable the other services. 

#### Getting started with firewalld

firewalld

firewalld is a firewall service daemon that provides a dynamic customizable host-based firewall with a D-Bus interface. Being dynamic, it enables creating, changing, and deleting the rules without the necessity to restart the firewall daemon each time the rules are changed.

firewalld uses the concepts of zones and services, that simplify the traffic management. Zones are predefined sets of rules. Network interfaces and sources can be assigned to a zone. The traffic allowed depends on the network your computer is connected to and the security level this network is assigned. Firewall services are predefined rules that cover all necessary settings to allow incoming traffic for a specific service and they apply within a zone.

Services use one or more ports or addresses for network communication. Firewalls filter communication based on ports. To allow network traffic for a service, its ports must be open. firewalld blocks all traffic on ports that are not explicitly set as open. Some zones, such as trusted, allow all traffic by default.

Additional resources

    firewalld(1) man page 

#### Zones

⁠firewalld can be used to separate networks into different zones according to the level of trust that the user has decided to place on the interfaces and traffic within that network. A connection can only be part of one zone, but a zone can be used for many network connections.

NetworkManager notifies firewalld of the zone of an interface. You can assign zones to interfaces with:

- NetworkManager
- firewall-config tool
- firewall-cmd command-line tool
- The RHEL web console 

The latter three can only edit the appropriate NetworkManager configuration files. If you change the zone of the interface using the web console, firewall-cmd or firewall-config, the request is forwarded to NetworkManager and is not handled by ⁠firewalld.

The predefined zones are stored in the /usr/lib/firewalld/zones/ directory and can be instantly applied to any available network interface. These files are copied to the /etc/firewalld/zones/ directory only after they are modified. The default settings of the predefined zones are as follows:

**block**
    Any incoming network connections are rejected with an icmp-host-prohibited message for IPv4 and icmp6-adm-prohibited for IPv6. Only network connections initiated from within the system are possible. 

**dmz**
    For computers in your demilitarized zone that are publicly-accessible with limited access to your internal network. Only selected incoming connections are accepted. 

**drop**
    Any incoming network packets are dropped without any notification. Only outgoing network connections are possible. 

**external**
    For use on external networks with masquerading enabled, especially for routers. You do not trust the other computers on the network to not harm your computer. Only selected incoming connections are accepted. 

**home**
    For use at home when you mostly trust the other computers on the network. Only selected incoming connections are accepted. 

**internal**
    For use on internal networks when you mostly trust the other computers on the network. Only selected incoming connections are accepted. 

**public**
    For use in public areas where you do not trust other computers on the network. Only selected incoming connections are accepted. 

**trusted**
    All network connections are accepted. 

**work**
    For use at work where you mostly trust the other computers on the network. Only selected incoming connections are accepted. 

One of these zones is set as the default zone. When interface connections are added to `NetworkManager`, they are assigned to the default zone. On installation, the default zone in `firewalld` is set to be the `public` zone. The default zone can be changed.

**Note**

The network zone names should be self-explanatory and to allow users to quickly make a reasonable decision. To avoid any security problems, review the default zone configuration and disable any unnecessary services according to your needs and risk assessments.

**Additional resources**

    firewalld.zone(5) man page 

#### Predefined services

A service can be a list of local ports, protocols, source ports, and destinations, as well as a list of firewall helper modules automatically loaded if a service is enabled. Using services saves users time because they can achieve several tasks, such as opening ports, defining protocols, enabling packet forwarding and more, in a single step, rather than setting up everything one after another.

Service configuration options and generic file information are described in the firewalld.service(5) man page. The services are specified by means of individual XML configuration files, which are named in the following format: service-name.xml. Protocol names are preferred over service or application names in firewalld.

Services can be added and removed using the graphical firewall-config tool, firewall-cmd, and firewall-offline-cmd.

Alternatively, you can edit the XML files in the /etc/firewalld/services/ directory. If a service is not added or changed by the user, then no corresponding XML file is found in /etc/firewalld/services/. The files in the /usr/lib/firewalld/services/ directory can be used as templates if you want to add or change a service.

**Additional resources**

    firewalld.service(5) man page 

------

#### Viewing the current status of firewalld

```
To see the status of the service:
# firewall-cmd --state

Or use the systemctl status sub-command:
# systemctl status firewalld
firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor pr
   Active: active (running) since Mon 2017-12-18 16:05:15 CET; 50min ago
     Docs: man:firewalld(1)
 Main PID: 705 (firewalld)
    Tasks: 2 (limit: 4915)
   CGroup: /system.slice/firewalld.service
           └─705 /usr/bin/python3 -Es /usr/sbin/firewalld --nofork --nopid
```

Viewing current firewalld settings

With the CLI client, it is possible to get different views of the current firewall settings. The --list-all option shows a complete overview of the firewalld settings.

firewalld uses zones to manage the traffic. If a zone is not specified by the --zone option, the command is effective in the default zone assigned to the active network interface and connection.


To list all the relevant information for the default zone:

```
# firewall-cmd --list-all
public
  target: default
  icmp-block-inversion: no
  interfaces:
  sources:
  services: ssh dhcpv6-client
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

To specify the zone for which to display the settings, add the --zone=zone-name argument to the firewall-cmd --list-all command, for example:

```
# firewall-cmd --list-all --zone=home
home
  target: default
  icmp-block-inversion: no
  interfaces:
  sources:
  services: ssh mdns samba-client dhcpv6-client
... [trimmed for clarity]
```

To list all the relevant information for all zones:

```
# firewall-cmd --list-all-zones
```

To see the settings for particular information, such as services or ports, use a specific option. See the firewalld manual pages or get a list of the options using the command help:

```
# firewall-cmd --help
Usage: firewall-cmd [OPTIONS...]

General Options
  -h, --help           Prints a short help text and exists
  -V, --version        Print the version string of firewalld
  -q, --quiet          Do not print status messages

Status Options
  --state              Return and print firewalld state
  --reload             Reload firewall and keep state information
... [trimmed for clarity]
```

For example, to see which services are allowed in the current zone:

```
# firewall-cmd --list-services

ssh dhcpv6-client
```

**Note**

Listing the settings for a certain subpart using the CLI tool can sometimes be difficult to interpret. For example, you allow the SSH service and firewalld opens the necessary port (22) for the service. Later, if you list the allowed services, the list shows the SSH service, but if you list open ports, it does not show any. Therefore, it is recommended to use the --list-all option to make sure you receive a complete information. 

------

#### Starting firewalld

```
To start firewalld, enter the following command as root:
# systemctl unmask firewalld
# systemctl start firewalld

To ensure firewalld starts automatically at system start, enter the following command as root:
# systemctl enable firewalld
```

------

#### Stopping firewalld

```
To stop firewalld, enter the following command as root:
# systemctl stop firewalld

To prevent firewalld from starting automatically at system start:
# systemctl disable firewalld

To make sure firewalld is not started by accessing the firewalld D-Bus interface and also if other services require firewalld:
# systemctl mask firewalld
```

------

#### Runtime and permanent settings

The configuration is separated into the runtime and the permanent configuration.

**Runtime Configuration**

The runtime configuration is the actual effective configuration and applied to the firewall in the kernel. At firewalld service start the permanent configuration becomes the runtime configuration. Changes in the runtime configuration are not automatically saved to the permanent configuration.

The runtime configuration will be lost with a firewalld service stop. A firewalld reload will replace the runtime configuration by the permanent configuration. Changed zone bindings will be restored after the reload.

**Permanent Configuration**

The permanent configuration is stored in configuration files and will be loaded and become new runtime configuration with every machine boot or service reload/restart.

If you set the rules while firewalld is running using only the --permanent option, they do not become effective before firewalld is restarted. However, restarting firewalld closes all open ports and stops the networking traffic.

**Runtime to Permanent**

Using the CLI, you do not modify the firewall settings in both modes at the same time. You only modify either runtime or permanent mode. To modify the firewall settings in the permanent mode, use the --permanent option with the firewall-cmd command.

```
# firewall-cmd --permanent <other options>

Without this option, the command modifies runtime mode.

To change settings in both modes, you can use two methods:

Change runtime settings and then make them permanent as follows:
# firewall-cmd <other options>
# firewall-cmd --runtime-to-permanent

Set permanent settings and reload the settings into runtime mode:
# firewall-cmd --permanent <other options>
# firewall-cmd --reload

The first method allows you to test the settings before you apply them to the permanent mode.
```

**Note**

It is possible, especially on remote systems, that an incorrect setting results in a user locking themselves out of a machine. To prevent such situations, use the --timeout option. After a specified amount of time, any change reverts to its previous state. Using this options excludes the --permanent option.

```
For example, to add the SSH service for 15 minutes:
# firewall-cmd --add-service=ssh --timeout 15m
```

-----

#### Verifying the permanent firewalld configuration

In certain situations, for example after manually editing firewalld configuration files, administrators want to verify that the changes are correct. This section describes how to verify the permanent configuration of the firewalld service.

```
Verify the permanent configuration of the firewalld service:
# firewall-cmd --check-config
success

If the permanent configuration is valid, the command returns success. In other cases, the command returns an error with further details, such as the following:
# firewall-cmd --check-config
Error: INVALID_PROTOCOL: 'public.xml': 'tcpx' not from {'tcp'|'udp'|'sctp'|'dccp'}
```

------

#### Directories

firewalld supports two configuration directories:

**Default and Fallback Configuration**

The directory /usr/lib/firewalld contains the default and fallback configuration provided by firewalld for icmptypes, services and zones. The files provided with the firewalld package should not get changed and the changes are gone with an update of the firewalld package. Additional icmptypes, services and zones can be provided with packages or by creating files.

**System Specific Configuration**

The system or user configuration stored in /etc/firewalld is either created by the system administrator or by customization with the configuration interface of firewalld or by hand. The files will overload the default configuration files.

To manually change settings of pre-defined icmptypes, zones or services, copy the file from the default configuration directory to the corresponding directory in the system configuration directory and change it accordingly.

If there is no /etc/firewalld directory of if it there is no configuration in there, firewalld will start using the default configuration and default settings for firewalld.conf.

------

#### Controlling network traffic using firewalld

**Disabling all traffic in case of emergency using CLI**

In an emergency situation, such as a system attack, it is possible to disable all network traffic and cut off the attacker.

```
To immediately disable networking traffic, switch panic mode on:
# firewall-cmd --panic-on
```

**Important**

Enabling panic mode stops all networking traffic. From this reason, it should be used only when you have the physical access to the machine or if you are logged in using a serial console.

```
Switching off panic mode reverts the firewall to its permanent settings. To switch panic mode off:
# firewall-cmd --panic-off

To see whether panic mode is switched on or off, use:
# firewall-cmd --query-panic
```

**Controlling traffic with predefined services using CLI**

The most straightforward method to control traffic is to add a predefined service to firewalld. This opens all necessary ports and modifies other settings according to the service definition file.

```
Check that the service is already allowed:
# firewall-cmd --list-services
cockpit dhcpv6-client ssh

List all predefined services:
# firewall-cmd --get-services
RH-Satellite-6 amanda-client amanda-k5-client bacula bacula-client bitcoin bitcoin-rpc bitcoin-testnet bitcoin-testnet-rpc ceph ceph-mon cfengine condor-collector ctdb dhcp dhcpv6 dhcpv6-client dns docker-registry ...
[trimmed for clarity]

Remove the service from the allowed services:
# firewall-cmd --remove-service=<service-name>

Add the service to the allowed services:
# firewall-cmd --add-service=<service-name>

Make the new settings persistent:
# firewall-cmd --runtime-to-permanent

Reload the permanent configuration:
# firewall-cmd --reload
```

**Defining new services**

There are different ways to define a new service. A new service will only be visible in permanent configuration after it has been defined. To make it active in the runtime environment you need to reload firewalld. Services can be defined and undefined using the graphical firewall-config tool, firewall-cmd, and firewall-offline-cmd. If a service is not defined or changed by the user, then no corresponding XML file are found in /etc/firewalld/services/. The files /usr/lib/firewalld/services/ can be used as templates if you want to add or change a service.

**Note**

Service names must be alphanumeric and can, additionally, include only _ (underscore) and - (dash) characters.

To define a new service in a terminal, use firewall-cmd, or firewall-offline-cmd in case of not active firewalld.

**With firewall-cmd**

```
To define a new and empty service, use the --new-service altogether with the --permanent option:

# firewall-cmd --permanent --new-service=myservice

Configure the service:

# firewall-cmd --permanent --service=myservice --set-description=description
# firewall-cmd --permanent --service=myservice --set-short=description
# firewall-cmd --permanent --service=myservice --add-port=portid[-portid]/protocol
# firewall-cmd --permanent --service=myservice --add-protocol=protocol
# firewall-cmd --permanent --service=myservice --add-source-port=portid[-portid]/protocol
# firewall-cmd --permanent --service=myservice --add-module=module
# firewall-cmd --permanent --service=myservice --set-destination=ipv:address[/mask]

Alternatively, you can define a new service using an existing file:
# firewall-cmd --permanent --new-service-from-file=myservice.xml

This defines a new service using all the settings from the file including the service name.

# firewall-cmd --permanent --new-service-from-file=myservice.xml --name=mynewservice

This defines a new service using the service settings from the file. The new service will have the name mynewservice.
```

**With firewall-offline-cmd**

```
To define a new and empty service, use the --new-service option:
# firewall-offline-cmd --new-service=myservice

Configure the service:
# firewall-offline-cmd --service=myservice --set-description=description
# firewall-offline-cmd --service=myservice --set-short=description
# firewall-offline-cmd --service=myservice --add-port=portid[-portid]/protocol
# firewall-offline-cmd --service=myservice --add-protocol=protocol
# firewall-offline-cmd --service=myservice --add-source-port=portid[-portid]/protocol
# firewall-offline-cmd --service=myservice --add-module=module
# firewall-offline-cmd --service=myservice --set-destination=ipv:address[/mask]

Alternatively, you can define a new service using an existing file:
# firewall-offline-cmd --new-service-from-file=myservice.xml

This defines a new service using all settings from the file including the service name.
# firewall-offline-cmd --new-service-from-file=myservice.xml --name=mynewservice

This defines a new service using the service settings from the file. But the new service will have the name mynewservice.
```

**Copy a file in the services directory in /etc/firewalld**

As root copy the file:

```
# cp myservice.xml /etc/firewalld/services
```

After you have copied the file into /etc/firewalld/services it takes about 5 seconds till the new service will be visible in firewalld.

**Place a file in the services directory in /usr/lib/firewalld**

This is how a package or system service could add a new service to firewalld. The benefit of placing the service into /usr/lib/firewalld/services is that the admin or user can modify the service and that they could go back to the original service easily by loading the defaults of the service. Then the firewalld created and modified copy in /etc/firewalld/services will be renamed to <service>.xml.old and the original service in /usr/lib/firewalld/services will be used again. The original service will be effective in the runtime environment only after a reload.

A package that places a service in the /usr/lib/firewalld/services directory should require the firewalld package or sub package that is providing the path. In an RPM based distribution that is using or that bases on the firewalld provided spec file this package is firewalld-filesystem.

**Controlling ports using CLI**

Ports are logical devices that enable an operating system to receive and distinguish network traffic and forward it accordingly to system services. These are usually represented by a daemon that listens on the port, that is it waits for any traffic coming to this port.

Normally, system services listen on standard ports that are reserved for them. The httpd daemon, for example, listens on port 80. However, system administrators by default configure daemons to listen on different ports to enhance security or for other reasons. 

**Opening a port**

Through open ports, the system is accessible from the outside, which represents a security risk. Generally, keep ports closed and only open them if they are required for certain services.

```
To get a list of open ports in the current zone:

List all allowed ports:
# firewall-cmd --list-ports

Add a port to the allowed ports to open it for incoming traffic:
# firewall-cmd --add-port=port-number/port-type

Make the new settings persistent:
# firewall-cmd --runtime-to-permanent
```

The port types are either tcp, udp, sctp, or dccp. The type must match the type of network communication. 

**Closing a port**

When an open port is no longer needed, close that port in firewalld. It is highly recommended to close all unnecessary ports as soon as they are not used because leaving a port open represents a security risk.

To close a port, remove it from the list of allowed ports:

```
List all allowed ports:
# firewall-cmd --list-ports
[WARNING]
====
This command will only give you a list of ports that have been opened as ports. You will not be able to see any open ports that have been opened as a service. Therefore, you should consider using the --list-all option instead of --list-ports.
====

Remove the port from the allowed ports to close it for the incoming traffic:
# firewall-cmd --remove-port=port-number/port-type

Make the new settings persistent:
# firewall-cmd --runtime-to-permanent
```

------

#### Working with firewalld zones

Zones represent a concept to manage incoming traffic more transparently. The zones are connected to networking interfaces or assigned a range of source addresses. You manage firewall rules for each zone independently, which enables you to define complex firewall settings and apply them to the traffic. 

**Listing zones**

To see which zones are available on your system:

```
# firewall-cmd --get-zones
```

The firewall-cmd --get-zones command displays all zones that are available on the system, but it does not show any details for particular zones.

To see detailed information for all zones:

```
# firewall-cmd --list-all-zones
```

To see detailed information for a specific zone:

```
# firewall-cmd --zone=zone-name --list-all
```

**Modifying firewalld settings for a certain zone**

To work in a different zone, use the --zone=zone-name option. For example, to allow the SSH service in the zone public: 

```
# firewall-cmd --add-service=ssh --zone=public
```

**Changing the default zone**

System administrators assign a zone to a networking interface in its configuration files. If an interface is not assigned to a specific zone, it is assigned to the default zone. After each restart of the firewalld service, firewalld loads the settings for the default zone and makes it active.

To set up the default zone:

Display the current default zone:

```
# firewall-cmd --get-default-zone
```

Set the new default zone:

```
# firewall-cmd --set-default-zone zone-name
```

**Note**

Following this procedure, the setting is a permanent setting, even without the --permanent option. 

**Assigning a network interface to a zone**

It is possible to define different sets of rules for different zones and then change the settings quickly by changing the zone for the interface that is being used. With multiple interfaces, a specific zone can be set for each of them to distinguish traffic that is coming through them.

To assign the zone to a specific interface:

List the active zones and the interfaces assigned to them:

```
# firewall-cmd --get-active-zones
```

Assign the interface to a different zone:

```
# firewall-cmd --zone=zone_name --change-interface=interface_name --permanent
```

**Assigning a zone to a connection using nmcli**

This procedure describes how to add a firewalld zone to a NetworkManager connection using the nmcli utility.

Assign the zone to the NetworkManager connection profile:

```
# nmcli connection modify profile connection.zone zone_name
```

Reload the connection:

```
# nmcli connection up profile
```

**Manually assigning a zone to a network connection in an ifcfg file**

When the connection is managed by NetworkManager, it must be aware of a zone that it uses. For every network connection, a zone can be specified, which provides the flexibility of various firewall settings according to the location of the computer with portable devices. Thus, zones and settings can be specified for different locations, such as company or home.

To set a zone for a connection, edit the /etc/sysconfig/network-scripts/ifcfg-connection_name file and add a line that assigns a zone to this connection:

```
ZONE=zone_name
```

**Creating a new zone**

To use custom zones, define a new zone and use it just like a predefined zone. New zones require the --permanent option, otherwise the command does not work.

```
To define a new zone:

Define a new zone:
# firewall-cmd --new-zone=zone-name

Check if the new zone is added to your permanent settings:
# firewall-cmd --get-zones

Make the new settings persistent:
# firewall-cmd --runtime-to-permanent
```

**Zone configuration files**

Zones can also be created using a zone configuration file. This approach can be helpful when you need to create a new zone, but want to reuse the settings from a different zone and only alter them a little.

A firewalld zone configuration file contains the information for a zone. These are the zone description, services, ports, protocols, icmp-blocks, masquerade, forward-ports and rich language rules in an XML file format. The file name has to be zone-name.xml where the length of zone-name is currently limited to 17 chars. The zone configuration files are located in the /usr/lib/firewalld/zones/ and /etc/firewalld/zones/ directories.

The following example shows a configuration that allows one service (SSH) and one port range, for both the TCP and UDP protocols:

```
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>My zone</short>
  <description>Here you can describe the characteristic features of the zone.</description>
  <service name="ssh"/>
  <port port="1025-65535" protocol="tcp"/>
  <port port="1025-65535" protocol="udp"/>
</zone>
```

To change settings for that zone, add or remove sections to add ports, forward ports, services, and so on.

**Additional resources**

    For more information, see the firewalld.zone manual pages. 

**Using zone targets to set default behavior for incoming traffic**

For every zone, you can set a default behavior that handles incoming traffic that is not further specified. Such behaviour is defined by setting the target of the zone. There are four options - default, ACCEPT, REJECT, and DROP. By setting the target to ACCEPT, you accept all incoming packets except those disabled by a specific rule. If you set the target to REJECT or DROP, you disable all incoming packets except those that you have allowed in specific rules. When packets are rejected, the source machine is informed about the rejection, while there is no information sent when the packets are dropped.

```
To set a target for a zone:

List the information for the specific zone to see the default target:
$ firewall-cmd --zone=zone-name --list-all

Set a new target in the zone:
# firewall-cmd --permanent --zone=zone-name --set-target=<default|ACCEPT|REJECT|DROP>
```

**Using zones to manage incoming traffic depending on a source**

You can use zones to manage incoming traffic based on its source. That enables you to sort incoming traffic and route it through different zones to allow or disallow services that can be reached by that traffic.

If you add a source to a zone, the zone becomes active and any incoming traffic from that source will be directed through it. You can specify different settings for each zone, which is applied to the traffic from the given sources accordingly. You can use more zones even if you only have one network interface. 

**Adding a source**

To route incoming traffic into a specific source, add the source to that zone. The source can be an IP address or an IP mask in the Classless Inter-domain Routing (CIDR) notation.

**Note**

In case you add multiple zones with an overlapping network range, they are ordered alphanumerically by zone name and only the first one is considered.

```
To set the source in the current zone:
# firewall-cmd --add-source=<source>

To set the source IP address for a specific zone:
# firewall-cmd --zone=zone-name --add-source=<source>
```

The following procedure allows all incoming traffic from 192.168.2.15 in the trusted zone:

```
List all available zones:
# firewall-cmd --get-zones

Add the source IP to the trusted zone in the permanent mode:
# firewall-cmd --zone=trusted --add-source=192.168.2.15

Make the new settings persistent:
# firewall-cmd --runtime-to-permanent
```

**Removing a source**

Removing a source from the zone cuts off the traffic coming from it.

```
List allowed sources for the required zone:
# firewall-cmd --zone=zone-name --list-sources

Remove the source from the zone permanently:
# firewall-cmd --zone=zone-name --remove-source=<source>

Make the new settings persistent:
# firewall-cmd --runtime-to-permanent
```

**Adding a source port**

To enable sorting the traffic based on a port of origin, specify a source port using the --add-source-port option. You can also combine this with the --add-source option to limit the traffic to a certain IP address or IP range.

```
To add a source port:
# firewall-cmd --zone=zone-name --add-source-port=<port-name>/<tcp|udp|sctp|dccp>
```

**Removing a source port**

By removing a source port you disable sorting the traffic based on a port of origin.

```
To remove a source port:
# firewall-cmd --zone=zone-name --remove-source-port=<port-name>/<tcp|udp|sctp|dccp>
```

**Using zones and sources to allow a service for only a specific domain**

To allow traffic from a specific network to use a service on a machine, use zones and source. The following procedure allows traffic from 192.168.1.0/24 to be able to reach the HTTP service while any other traffic is blocked.

```
List all available zones:

# firewall-cmd --get-zones
block dmz drop external home internal public trusted work

Add the source to the trusted zone to route the traffic originating from the source through the zone:
# firewall-cmd --zone=trusted --add-source=192.168.1.0/24

Add the http service in the trusted zone:
# firewall-cmd --zone=trusted --add-service=http

Make the new settings persistent:
# firewall-cmd --runtime-to-permanent

Check that the trusted zone is active and that the service is allowed in it:

# firewall-cmd --zone=trusted --list-all
trusted (active)
target: ACCEPT
sources: 192.168.1.0/24
services: http
```

**Configuring traffic accepted by a zone based on a protocol**

You can allow incoming traffic to be accepted by a zone based on a protocol. All traffic using the specified protocol is accepted by a zone, in which you can apply further rules and filtering. 

**Adding a protocol to a zone**

By adding a protocol to a certain zone, you allow all traffic with this protocol to be accepted by this zone.

```
To add a protocol to a zone:
# firewall-cmd --zone=zone-name --add-protocol=port-name/tcp|udp|sctp|dccp|igmp
```

**Note**

To receive multicast traffic, use the igmp value with the --add-protocol option. 
**Removing a protocol from a zone**

By removing a protocol from a certain zone, you stop accepting all traffic based on this protocol by the zone.

```
To remove a protocol from a zone:
# firewall-cmd --zone=zone-name --remove-protocol=port-name/tcp|udp|sctp|dccp|igmp
```

------

#### Configuring IP address masquerading

The following procedure describes how to enable IP masquerading on your system. IP masquerading hides individual machines behind a gateway when accessing the Internet.

```
To check if IP masquerading is enabled (for example, for the external zone), enter the following command as root:
# firewall-cmd --zone=external --query-masquerade

The command prints yes with exit status 0 if enabled. It prints no with exit status 1 otherwise. If zone is omitted, the default zone will be used.

To enable IP masquerading, enter the following command as root:
# firewall-cmd --zone=external --add-masquerade

To make this setting persistent, repeat the command adding the --permanent option. 

To disable IP masquerading, enter the following command as root:
# firewall-cmd --zone=external --remove-masquerade --permanent
```

------

#### Port forwarding

Redirecting ports using this method only works for IPv4-based traffic. For IPv6 redirecting setup, you must use rich rules. 

**Adding a port to redirect**

Using firewalld, you can set up ports redirection so that any incoming traffic that reaches a certain port on your system is delivered to another internal port of your choice or to an external port on another machine.

**Prerequisites**

    Before you redirect traffic from one port to another port, or another address, you have to know three things: which port the packets arrive at, what protocol is used, and where you want to redirect them. 

To redirect a port to another port:

```
# firewall-cmd --add-forward-port=port=port-number:proto=tcp|udp|sctp|dccp:toport=port-number
```

To redirect a port to another port at a different IP address:

```
Add the port to be forwarded:
# firewall-cmd --add-forward-port=port=port-number:proto=tcp|udp:toport=port-number:toaddr=IP

Enable masquerade:
# firewall-cmd --add-masquerade
```

Redirecting TCP port 80 to port 88 on the same machine

Follow the steps to redirect the TCP port 80 to port 88.

```
Redirect the port 80 to port 88 for TCP traffic:
# firewall-cmd --add-forward-port=port=80:proto=tcp:toport=88

Make the new settings persistent:
# firewall-cmd --runtime-to-permanent

Check that the port is redirected:
# firewall-cmd --list-all
```

Removing a redirected port

To remove a redirected port:

```
# firewall-cmd --remove-forward-port=port=port-number:proto=<tcp|udp>:toport=port-number:toaddr=<IP>
```
To remove a forwarded port redirected to a different address, use the following procedure.

```
Remove the forwarded port:
# firewall-cmd --remove-forward-port=port=port-number:proto=<tcp|udp>:toport=port-number:toaddr=<IP>

Disable masquerade:
# firewall-cmd --remove-masquerade
```

Removing TCP port 80 forwarded to port 88 on the same machine

```
To remove the port redirection:

List redirected ports:
~]# firewall-cmd --list-forward-ports
port=80:proto=tcp:toport=88:toaddr=

Remove the redirected port from the firewall::
~]# firewall-cmd  --remove-forward-port=port=80:proto=tcp:toport=88:toaddr=

Make the new settings persistent:
~]# firewall-cmd --runtime-to-permanent
```

------

#### Managing ICMP requests

The Internet Control Message Protocol (ICMP) is a supporting protocol that is used by various network devices to send error messages and operational information indicating a connection problem, for example, that a requested service is not available. ICMP differs from transport protocols such as TCP and UDP because it is not used to exchange data between systems.

Unfortunately, it is possible to use the ICMP messages, especially echo-request and echo-reply, to reveal information about your network and misuse such information for various kinds of fraudulent activities. Therefore, firewalld enables blocking the ICMP requests to protect your network information. 

**Listing and blocking ICMP requests**

Listing ICMP requests

The ICMP requests are described in individual XML files that are located in the /usr/lib/firewalld/icmptypes/ directory. The firewall-cmd command controls the ICMP requests manipulation. 

```
--get-icmptypes		/* To list all available ICMP types.

/* The ICMP request can be used by IPv4, IPv6, or by both protocols. To see for which protocol the ICMP request is used.
--info-icmptype=<icmptype>

/* To see if an ICMP request is currently blocked. blocked -> yes.
--query-icmp-block=<icmptype>
    
--add-icmp-block=<icmptype>	/* block an ICMP request.
--remove-icmp-block=<icmptype>	/* remove the block for an ICMP request.

/* The block inversion inverts the setting of the ICMP requests blocks. 
--add-icmp-block-inversion	

--remove-icmp-block-inversion	/* Remove the ICMP block inversion
```

Note: Blocking the ICMP requests should be considered carefully, because it can cause communication problems, especially with IPv6 traffic.

**Blocking ICMP requests without providing any information at all**

Normally, if you block ICMP requests, clients know that you are blocking it. So, a potential attacker who is sniffing for live IP addresses is still able to see that your IP address is online. To hide this information completely, you have to drop all ICMP requests. 

------

#### Setting and controlling IP sets using firewalld

To see the list of IP set types supported by firewalld, enter the following command as root.

```
# firewall-cmd --get-ipset-types
hash:ip hash:ip,mark hash:ip,port hash:ip,port,ip hash:ip,port,net hash:mac hash:net hash:net,iface hash:net,net hash:net,port hash:net,port,net
```

**Configuring IP set options using CLI**

IP sets can be used in firewalld zones as sources and also as sources in rich rules. In Red Hat Enterprise Linux, the preferred method is to use the IP sets created with firewalld in a direct rule.

```
To list the IP sets known to firewalld in the permanent environment, use the following command as root:
# firewall-cmd --permanent --get-ipsets

To add a new IP set, use the following command using the permanent environment as root:
# firewall-cmd --permanent --new-ipset=test --type=hash:net
success

The previous command creates a new IP set with the name test and the hash:net type for IPv4. To create an IP set for use with IPv6, add the --option=family=inet6 option. To make the new setting effective in the runtime environment, reload firewalld.

List the new IP set with the following command as root:
# firewall-cmd --permanent --get-ipsets
test

To get more information about the IP set, use the following command as root:
# firewall-cmd --permanent --info-ipset=test
test
type: hash:net
options:
entries:

Note that the IP set does not have any entries at the moment.

To add an entry to the test IP set, use the following command as root:
# firewall-cmd --permanent --ipset=test --add-entry=192.168.0.1
success

The previous command adds the IP address 192.168.0.1 to the IP set.

To get the list of current entries in the IP set, use the following command as root:
# firewall-cmd --permanent --ipset=test --get-entries
192.168.0.1

Generate a file containing a list of IP addresses, for example:
# cat > iplist.txt <<EOL
192.168.0.2
192.168.0.3
192.168.1.0/24
192.168.2.254
EOL

The file with the list of IP addresses for an IP set should contain an entry per line. Lines starting with a hash, a semi-colon, or empty lines are ignored.

To add the addresses from the iplist.txt file, use the following command as root:
# firewall-cmd --permanent --ipset=test --add-entries-from-file=iplist.txt
success

To see the extended entries list of the IP set, use the following command as root:
# firewall-cmd --permanent --ipset=test --get-entries
192.168.0.1
192.168.0.2
192.168.0.3
192.168.1.0/24
192.168.2.254

To remove the addresses from the IP set and to check the updated entries list, use the following commands as root:
# firewall-cmd --permanent --ipset=test --remove-entries-from-file=iplist.txt
success
# firewall-cmd --permanent --ipset=test --get-entries
192.168.0.1

You can add the IP set as a source to a zone to handle all traffic coming in from any of the addresses listed in the IP set with a zone. For example, to add the test IP set as a source to the drop zone to drop all packets coming from all entries listed in the test IP set, use the following command as root:

# firewall-cmd --permanent --zone=drop --add-source=ipset:test
success
```

The ipset: prefix in the source shows firewalld that the source is an IP set and not an IP address or an address range. 

Only the creation and removal of IP sets is limited to the permanent environment, all other IP set options can be used also in the runtime environment without the --permanent option.

**Warning**

Red Hat does not recommend using IP sets that are not managed through firewalld. To use such IP sets, a permanent direct rule is required to reference the set, and a custom service must be added to create these IP sets. This service needs to be started before firewalld starts, otherwise firewalld is not able to add the direct rules using these sets. You can add permanent direct rules with the /etc/firewalld/direct.xml file. 

------

#### Prioritizing rich rules

By default, rich rules are organized based on their rule action. For example, deny rules have precedence over allow rules. The priority parameter in rich rules provides administrators fine-grained control over rich rules and their execution order. 

**How the priority parameter organizes rules into different chains**

You can set the priority parameter in a rich rule to any number between -32768 and 32767, and lower values have higher precedence.

The firewalld service organizes rules based on their priority value into different chains:

Priority lower than 0: the rule is redirected into a chain with the _pre suffix.
Priority higher than 0: the rule is redirected into a chain with the _post suffix.
Priority equals 0: based on the action, the rule is redirected into a chain with the _log, _deny, or _allow the action. 

Inside these sub-chains, firewalld sorts the rules based on their priority value. 

**Setting the priority of a rich rule**

The procedure describes an example of how to create a rich rule that uses the priority parameter to log all traffic that is not allowed or denied by other rules. You can use this rule to flag unexpected traffic.

```
Add a rich rule with a very low precedence to log all traffic that has not been matched by other rules:
# firewall-cmd --add-rich-rule='rule priority=32767 log prefix="UNEXPECTED: " limit value="5/m"'

The command additionally limits the number of log entries to 5 per minute.

Optionally, display the nftables rule that the command in the previous step created:
# nft list chain inet firewalld filter_IN_public_post
table inet firewalld {
  chain filter_IN_public_post {
    log prefix "UNEXPECTED: " limit rate 5/minute
  }
}
```

------

#### Configuring firewall lockdown

Local applications or services are able to change the firewall configuration if they are running as root (for example, libvirt). With this feature, the administrator can lock the firewall configuration so that either no applications or only applications that are added to the lockdown allow list are able to request firewall changes. The lockdown settings default to disabled. If enabled, the user can be sure that there are no unwanted configuration changes made to the firewall by local applications or services. 

**Configuring lockdown using CLI**

```
To query whether lockdown is enabled, use the following command as root:
# firewall-cmd --query-lockdown

The command prints yes with exit status 0 if lockdown is enabled. It prints no with exit status 1 otherwise.

To enable lockdown, enter the following command as root:
# firewall-cmd --lockdown-on

To disable lockdown, use the following command as root:
# firewall-cmd --lockdown-off
```

**Configuring lockdown allowlist options using CLI**

The lockdown allowlist can contain commands, security contexts, users and user IDs. If a command entry on the allowlist ends with an asterisk "*", then all command lines starting with that command will match. If the "*" is not there then the absolute command including arguments must match.

The context is the security (SELinux) context of a running application or service. To get the context of a running application use the following command:

$ ps -e --context

That command returns all running applications. Pipe the output through the grep tool to get the application of interest. For example:

$ ps -e --context | grep example_program

To list all command lines that are in the allowlist, enter the following command as root:

# firewall-cmd --list-lockdown-whitelist-commands

To add a command command to the allowlist, enter the following command as root:

# firewall-cmd --add-lockdown-whitelist-command='/usr/bin/python3 -Es /usr/bin/command'

To remove a command command from the allowlist, enter the following command as root:

# firewall-cmd --remove-lockdown-whitelist-command='/usr/bin/python3 -Es /usr/bin/command'

To query whether the command command is in the allowlist, enter the following command as root:

# firewall-cmd --query-lockdown-whitelist-command='/usr/bin/python3 -Es /usr/bin/command'

The command prints yes with exit status 0 if true. It prints no with exit status 1 otherwise.

To list all security contexts that are in the allowlist, enter the following command as root:

# firewall-cmd --list-lockdown-whitelist-contexts

To add a context context to the allowlist, enter the following command as root:

# firewall-cmd --add-lockdown-whitelist-context=context

To remove a context context from the allowlist, enter the following command as root:

# firewall-cmd --remove-lockdown-whitelist-context=context

To query whether the context context is in the allowlist, enter the following command as root:

# firewall-cmd --query-lockdown-whitelist-context=context

Prints yes with exit status 0, if true, prints no with exit status 1 otherwise.

To list all user IDs that are in the allowlist, enter the following command as root:

# firewall-cmd --list-lockdown-whitelist-uids

To add a user ID uid to the allowlist, enter the following command as root:

# firewall-cmd --add-lockdown-whitelist-uid=uid

To remove a user ID uid from the allowlist, enter the following command as root:

# firewall-cmd --remove-lockdown-whitelist-uid=uid

To query whether the user ID uid is in the allowlist, enter the following command:

$ firewall-cmd --query-lockdown-whitelist-uid=uid

Prints yes with exit status 0, if true, prints no with exit status 1 otherwise.

To list all user names that are in the allowlist, enter the following command as root:

# firewall-cmd --list-lockdown-whitelist-users

To add a user name user to the allowlist, enter the following command as root:

# firewall-cmd --add-lockdown-whitelist-user=user

To remove a user name user from the allowlist, enter the following command as root:

# firewall-cmd --remove-lockdown-whitelist-user=user

To query whether the user name user is in the allowlist, enter the following command:

$ firewall-cmd --query-lockdown-whitelist-user=user

Prints yes with exit status 0, if true, prints no with exit status 1 otherwise. 

**Configuring lockdown allowlist options using configuration files**

The default allowlist configuration file contains the NetworkManager context and the default context of libvirt. The user ID 0 is also on the list.

<?xml version="1.0" encoding="utf-8"?>
	<whitelist>
	  <selinux context="system_u:system_r:NetworkManager_t:s0"/>
	  <selinux context="system_u:system_r:virtd_t:s0-s0:c0.c1023"/>
	  <user id="0"/>
	</whitelist>

Following is an example allowlist configuration file enabling all commands for the firewall-cmd utility, for a user called user whose user ID is 815:

<?xml version="1.0" encoding="utf-8"?>
	<whitelist>
	  <command name="/usr/libexec/platform-python -s /bin/firewall-cmd*"/>
	  <selinux context="system_u:system_r:NetworkManager_t:s0"/>
	  <user id="815"/>
	  <user name="user"/>
	</whitelist>

This example shows both user id and user name, but only one option is required. Python is the interpreter and is prepended to the command line. You can also use a specific command, for example:

/usr/bin/python3 /bin/firewall-cmd --lockdown-on

In that example, only the --lockdown-on command is allowed.

In Red Hat Enterprise Linux, all utilities are placed in the /usr/bin/ directory and the /bin/ directory is sym-linked to the /usr/bin/ directory. In other words, although the path for firewall-cmd when entered as root might resolve to /bin/firewall-cmd, /usr/bin/firewall-cmd can now be used. All new scripts should use the new location. But be aware that if scripts that run as root are written to use the /bin/firewall-cmd path, then that command path must be added in the allowlist in addition to the /usr/bin/firewall-cmd path traditionally used only for non-root users.

The * at the end of the name attribute of a command means that all commands that start with this string match. If the * is not there then the absolute command including arguments must match. 

------

#### Log for denied packets

With the LogDenied option in the firewalld, it is possible to add a simple logging mechanism for denied packets. These are the packets that are rejected or dropped. To change the setting of the logging, edit the /etc/firewalld/firewalld.conf file or use the command-line or GUI configuration tool.

If LogDenied is enabled, logging rules are added right before the reject and drop rules in the INPUT, FORWARD and OUTPUT chains for the default rules and also the final reject and drop rules in zones. The possible values for this setting are: all, unicast, broadcast, multicast, and off. The default setting is off. With the unicast, broadcast, and multicast setting, the pkttype match is used to match the link-layer packet type. With all, all packets are logged.

To list the actual LogDenied setting with firewall-cmd, use the following command as root:

# firewall-cmd --get-log-denied
off

To change the LogDenied setting, use the following command as root:

# firewall-cmd --set-log-denied=all
success

To change the LogDenied setting with the firewalld GUI configuration tool, start firewall-config, click the Options menu and select Change Log Denied. The LogDenied window appears. Select the new LogDenied setting from the menu and click OK. 

------

#### Related information

The following sources of information provide additional resources regarding firewalld.
Installed documentation

    firewalld(1) man page — describes command options for firewalld.
    firewalld.conf(5) man page — contains information to configure firewalld.
    firewall-cmd(1) man page — describes command options for the firewalld command-line client.
    firewall-config(1) man page — describes settings for the firewall-config tool.
    firewall-offline-cmd(1) man page — describes command options for the firewalld offline command-line client.
    firewalld.icmptype(5) man page — describes XML configuration files for ICMP filtering.
    firewalld.ipset(5) man page — describes XML configuration files for the firewalld IP sets.
    firewalld.service(5) man page — describes XML configuration files for firewalld service.
    firewalld.zone(5) man page — describes XML configuration files for firewalld zone configuration.
    firewalld.direct(5) man page — describes the firewalld direct interface configuration file.
    firewalld.lockdown-whitelist(5) man page — describes the firewalld lockdown allowlist configuration file.
    firewalld.richlanguage(5) man page — describes the firewalld rich language rule syntax.
    firewalld.zones(5) man page — general description of what zones are and how to configure them.
    firewalld.dbus(5) man page — describes the D-Bus interface of firewalld. 

Online documentation

    http://www.firewalld.org/ — firewalld home page. 






------





https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/configuring_and_managing_networking/index#general-rhel-networking-topics_configuring-and-managing-networking
