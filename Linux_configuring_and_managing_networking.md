## Chapter 1. General RHEL networking topics

https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/general-rhel-networking-topics_configuring-and-managing-networking

------

### Categories of network communication:

**IP networks**

Networks that communicate through IP addresses. Such as Ethernet, wireless networks, and VPN connections, etc.

**Non-IP networks**

Networks that are used to communicate through a lower layer rather than the transport layer. Note that these networks are rarely used. For example, InfiniBand is a non-IP network.

### DHCP transaction phases

The DHCP works in four phases: Discovery, Offer, Request, Acknowledgement, also called the DORA process. DHCP uses this process to provide IP addresses to clients.

**Discovery**

The DHCP client sends a message to discover the DHCP server in the network. This message is broadcasted at the network and data link layer.

**Offer**

The DHCP server receives messages from the client and offers an IP address to the DHCP client. This message is unicast at the data link layer but broadcast at the network layer.

**Request**

The DHCP client requests the DHCP server for the offered IP address. This message is unicast at the data link layer but broadcast at the network layer.

**Acknowledgment**

The DHCP server sends an acknowledgment to the DHCP client. This message is unicast at the data link layer but broadcast at the network layer. It is the final message of the DHCP DORA process.

### Legacy network scripts support in RHEL

By default, RHEL uses NetworkManager to configure and manage network connections, and the /usr/sbin/ifup and /usr/sbin/ifdown scripts use NetworkManager to process ifcfg files in the /etc/sysconfig/network-scripts/ directory.

The legacy scripts are deprecated in RHEL 8 and will be removed in a future major version of RHEL. If you still want to use the legacy network scripts, you can install it, like `sudo yum install network-scripts`.

### Selecting network configuration methods

Directly edit network configuration files `ifcfg` manually. Note that even if you edit the files directly, NetworkManager is the default on RHEL and processes these files. Only if you installed and enabled the deprecated legacy networking scrips, then these scripts process the ifcfg files.

- Using NetworkManager to configure network interfaces by using one of the following tools:

    - the command-line utility , nmcli.
    - the text user interface, nmtui.
    - the graphical user interface tools, GNOME GUI.

- To configure a network interface without using NetworkManager tools and applications:

    - edit the ifcfg files manually. Note that even if you edit the files directly, NetworkManager is the default on RHEL and processes these files. Only if you installed and enabled the deprecated legacy networking scrips, then these scripts process the ifcfg files.

- To configure the network settings when the root file system is not local:

    - use the kernel command-line.

------

## Chapter 2. Consistent network interface device naming

Red Hat Enterprise Linux 8 provides methods for consistent and predictable device naming for network interfaces. These features help locating and differentiating network interfaces.

The kernel assigns names to network interfaces by concatenating a fixed prefix and a number that increases as the kernel initialize the network devices. For instance, `eth0` would represent the first device being probed on start-up. However, these names do not necessarily correspond to labels on the chassis. Modern server platforms with multiple network adapters can encounter non-deterministic and counter-intuitive naming of these interfaces. This affects both network adapters embedded on the system board and add-in adapters.

In Red Hat Enterprise Linux 8, the `udev` device manager supports a number of different naming schemes. By default, udev assigns fixed names based on firmware, topology, and location information. This has the following advantages:

- Device names are fully predictable.

- Device names stay fixed even if you add or remove hardware, because no re-enumeration takes places.

- Defective hardware can be seamlessly replaced.

------

### Network interface device naming hierarchy

If consistent device naming is enabled, which is the default in Red Hat Enterprise Linux 8, the `udev` device manager generates device names based on the following schemes:

| Scheme | Description | Example |
| ------ | ----------- | ------- |
| 1      | Device names incorporate firmware or BIOS-provided index numbers for onboard devices. If this information is not available or applicable, `udev` uses scheme 2. | `eno1` |
| 2      | Device names incorporate firmware or BIOS-provided PCI Express (PCIe) hot plug slot index numbers. If this information is not available or applicable, udev uses scheme 3. | `ens1` |
| 3      | Device names incorporate the physical location of the connector of the hardware. If this information is not available or applicable, `udev` uses scheme 5. | `enp2s0` |
| 4      | Device names incorporate the MAC address. Red Hat Enterprise Linux does not use this scheme by default, but administrators can optionally use it. | `enx525400d5e0fb` |
| 5      | The traditional unpredictable kernel naming scheme. If `udev` cannot apply any of the other schemes, the device manager uses this scheme. | `eth0` |

By default, Red Hat Enterprise Linux selects the device name based on the NamePolicy setting in the /usr/lib/systemd/network/99-default.link file. The order of the values in NamePolicy is important. Red Hat Enterprise Linux uses the first device name that is both specified in the file and that udev generated.

If you manually configured udev rules to change the name of kernel devices, those rules take precedence.

------

### How the network device renaming works

By default, consistent device naming is enabled in Red Hat Enterprise Linux 8. The `udev` device manager processes different rules to rename the devices. The following list describes the order in which `udev` processes these rules and what actions these rules are responsible for:

1. The `/usr/lib/udev/rules.d/60-net.rules` file defines that the `/lib/udev/rename_device` helper utility searches for the HWADDR parameter in `/etc/sysconfig/network-scripts/ifcfg-*` files. If the value set in the variable matches the MAC address of an interface, the helper utility renames the interface to the name set in the `DEVICE` parameter of the file.

2. The `/usr/lib/udev/rules.d/71-biosdevname.rules` file defines that the `biosdevname` utility renames the interface according to its naming policy, provided that it was not renamed in the previous step.

3. The `/usr/lib/udev/rules.d/75-net-description.rules` file defines that `udev` examines the network interface device and sets the properties in `udev`-internal variables, that will be processed in the next step. Note that some of these properties might be undefined.

4. The `/usr/lib/udev/rules.d/80-net-setup-link.rules` file calls the `net_setup_link` `udev` built-in which then applies the policy. The following is the default policy that is stored in the `/usr/lib/systemd/network/99-default.link` file:

    ```
    [Link]
    NamePolicy=kernel database onboard slot path
    MACAddressPolicy=persistent
    ```

    With this policy, if the kernel uses a persistent name, `udev` does not rename the interface. If the kernel does not use a persistent name, udev renames the interface to the name provided by the hardware database of `udev`. If this database is not available, Red Hat Enterprise Linux falls back to the mechanisms described above.

    Alternatively, set the NamePolicy parameter in this file to mac for media access control (MAC) address-based interface names.

5. The `/usr/lib/udev/rules.d/80-net-setup-link.rules` file defines that `udev` renames the interface based on the `udev`-internal parameters in the following order:

    - ID_NET_NAME_ONBOARD
    - ID_NET_NAME_SLOT
    - ID_NET_NAME_PATH

    If one parameter is not set, udev uses the next one. If none of the parameters are set, the interface is not renamed.

Steps 3 and 4 implement the naming schemes 1 to 4 described in Section 2.1, “Network interface device naming hierarchy”.

**Additional resources**

- For details about setting custom prefixes for consistent naming, see Section 2.7, “Using prefixdevname for naming of Ethernet network interfaces”.

- For details about the `NamePolicy` parameter, see the `systemd.link(5)` man page.

------

### 2.3. Predictable network interface device names on the x86_64 platform explained

When the consistent network device name feature is enabled, the `udev` device manager creates the names of devices based on different criteria. This section describes the naming scheme when Red Hat Enterprise Linux 8 is installed on a x86_64 platform.

The interface name starts with a two-character prefix based on the type of interface:

- en for Ethernet
- wl for wireless LAN (WLAN)
- ww for wireless wide area network (WWAN)

Additionally, one of the following is appended to one of the above-mentioned prefix based on the schema the `udev` device manager applies:

- `o<on-board_index_number>`

- `s<hot_plug_slot_index_number>[f<function>][d<device_id>]`

    Note that all multi-function PCI devices have the `[f<function>]` number in the device name, including the function `0` device.

- `x<MAC_address>`

- `[P<domain_number>]p<bus>s<slot>[f<function>][d<device_id>]`

    The `[P<domain_number>]` part defines the PCI geographical location. This part is only set if the domain number is not `0`.

- `[P<domain_number>]p<bus>s<slot>[f<function>][u<usb_port>][…​][c<config>][i<interface>]`

    For USB devices, the full chain of port numbers of hubs is composed. If the name is longer than the maximum (15 characters), the name is not exported. If there are multiple USB devices in the chain, `udev` suppresses the default values for USB configuration descriptors (`c1`) and USB interface descriptors (`i0`).

------

### Disabling consistent interface device naming during the installation

This section describes how to disable consistent interface device naming during the installation.

**Warning**

Red Hat recommends not to disable consistent device naming. Disabling consistent device naming can cause different kind of problems. For example, if you add another network interface card to the system, the assignment of the kernel device names, such as eth0, is no longer fixed. Consequently, after a reboot, the Kernel can name the device differently.

**Procedure**

1. Boot the Red Hat Enterprise Linux 8 installation media.

2. In the boot manager, select `Install Red Hat Enterprise Linux 8`, and press the `Tab` key to edit the entry.

3. Append the `net.ifnames=0` parameter to the kernel command line:

`vmlinuz... net.ifnames=0`

4. Press Enter to start the installation.

**Additional resources**

- [Is it safe to set net.ifnames=0 in RHEL 7 and RHEL 8?][25401]

- [How to perform an in-place upgrade to RHEL 8 when using kernel NIC names on RHEL 7][25402]

[25401]: <https://access.redhat.com/solutions/2435891>
[25402]: <https://access.redhat.com/solutions/4067471>

------

### Disabling consistent interface device naming on an installed System

This section describes how to disable consistent interface device naming on a system that is already installed.

**Warning**

Red Hat recommends not to disable consistent device naming. Disabling consistent device naming can cause different kind of problems. For example, if you add another network interface card to the system, the assignment of the kernel device names, such as eth0, is no longer fixed. Consequently, after a reboot, the Kernel can name the device differently.

**Prerequisites**

- The system uses consistent interface device naming, which is the default.

**Procedure**

1. Edit the `/etc/default/grub` file and append the `net.ifnames=0` parameter to the `GRUB_CMDLINE_LINUX` variable:

`GRUB_CMDLINE_LINUX="... net.ifnames=0`

2. Rebuild the `grub.cfg` file:

- On a system with UEFI boot mode:

`# grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg`

- On a system with legacy boot mode:

`# grub2-mkconfig -o /boot/grub2/grub.cfg`

3. If you use interface names in configuration files or scripts, you must manually update them.

4. Reboot the host:

`# reboot`

------

### Using prefixdevname for naming of Ethernet network interfaces

This documentation describes how to set the prefixes for consistent naming of Ethernet network interfaces in case that you do not want to use the default naming scheme of such interfaces. However, Red Hat recommends to use the default naming scheme. For more details about this scheme, see Chapter 2, Consistent network interface device naming.

#### Introduction to prefixdevname

The `prefixdevname` tool is a udev helper utility that enables you to define your own prefix used for naming of the Ethernet network interfaces.

#### Limitations of prefixdevname

There are certain limitations for prefixes of Ethernet network interfaces.

The prefix that you choose must meet the following requirements:

- Be ASCII string
- Be alphanumeric string
- Be shorter than 16 characters

**Warning**

The prefix cannot conflict with any other well-known prefix used for network interface naming on Linux. Specifically, you cannot use these prefixes: eth, eno, ens, em.

#### Setting prefixdevname

The setting of the prefix with `prefixdevname` is done during system installation.

To set and activate the required prefix for your Ethernet network interfaces, use the following procedure.

**Procedure**

- Add the following string on the kernel command line:

`net.ifnames.prefix=<required prefix>`

**Warning**

Red Hat does not support the use of prefixdevname on already deployed systems.

After the prefix was once set, and the operating system was rebooted, the prefix is effective every time when a new network interface appears. The new device is assigned a name in the form of `<PREFIX><INDEX>`. For example, if your selected prefix is `net`, and the interfaces with `net0` and `net1` prefixes already exist on the system, the new interface is named `net2`. The `prefixdevname` utility then generates the new `.link` file in the `/etc/systemd/network` directory that applies the name to the interface with the MAC address that just appeared. The configuration is persistent across reboots.

------

### Related information

- See the udev(7) man page for details about the udev device manager.

Information for VPNs, mobile broadband and PPPoE connections is stored in /etc/NetworkManager/system-connections/.

------

## Chapter 3. Getting started with NetworkManager

By default, RHEL 8 uses NetworkManager to manage the network configuration and connections.

### 3.1. Benefits of using NetworkManager

NetwrokManager offers an API through D-Bus which allows to query and control network configuration and state. This makes Network management easier. for example, when it detects that there is no network configuration in a system but there are network devices, NetworkManager creates temporary connections to provide connectivity.

------

### 3.2. An overview of utilities and applications you can use to manage NetworkManager connections

There are several tools below to manage NetworkManager connections:

- nmcli: A command-line utility.
- nmtui: A curses-based text user interface (TUI). it belongs t0 NetworkManager-tui package.
- nm-connection-editor: A graphical user interface (GUI) for NetworkManager-related tasks.
- control-center: A GUI provided by the GNOME shell for desktop users. Note thatit supports less features than nm-connection-editor.
- The network connection icon in the GNOME shell: This icon represents network connection states and serves as visual indicator for the type of connection you are using.

------

### 3.3. Using NetworkManager dispatcher scripts

By default, the /etc/NetworkManager/dispatcher.d/ directory exists and NetworkManager runs scripts there, in alphabetical order. Each script must be an executable file owned by root and must have write permission only for the file owner.

**Additional resources**

- For an example of a dispatcher script, see the [How to write a NetworkManager dispatcher script to apply ethtool commands solution][3301].

[3301]: <https://access.redhat.com/solutions/2841131>

------

### 3.4. Loading manually-created ifcfg files into NetworkManager

Interface-specific information is stored in the ifcfg files in the `/etc/sysconfig/network-scripts/` directory. Information for VPNs, mobile broadband and PPPoE connections is stored in `/etc/NetworkManager/system-connections/`.

If you change the configuration files or update NetworkManager profile settings, NetworkManager does not implement those changes until you reconnect using those files.

for example,

1. To load a new configuration file:

`# sudo nmcli connection load /etc/sysconfig/network-scripts/ifcfg-connection_name`

2. If you updated a connection file that has already been loaded into NetworkManager, enter:

`# sudo nmcli connection up connection_name`

**Additional resources**

- `NetworkManager(8)` man page — Describes the network management daemon.

- `NetworkManager.conf(5)` man page — Describes the NetworkManager configuration file.

- `/usr/share/doc/initscripts/sysconfig.txt` — Describes ifcfg configuration files and their directives as understood by the legacy network service.

- `ifcfg(8)` man page — Describes briefly the ifcfg command.

------

## Chapter 4. Configuring NetworkManager to ignore certain devices

By default, NetworkManager manages all devices except the lo (loopback) device. However, you can set certain devices as unmanaged to configure that NetworkManager ignores these devices. With this setting, you can manually manage these devices, for example, using a script.

------

### 4.1. Permanently configuring a device as unmanaged in NetworkManager

You can configure devices as `unmanaged` based on several criteria, such as the interface name, MAC address, or device type. This procedure describes how to permanently set the `enp1s0` interface as `unmanaged` in NetworkManager.

To temporarily configure network devices as `unmanaged`, see Section 4.2, “Temporarily configuring a device as unmanaged in NetworkManager”.

**Procedure**

1. Optional: Display the list of devices to identify the device you want to set as `unmanaged`:

```
# nmcli device status
DEVICE  TYPE      STATE         CONNECTION
enp1s0  ethernet  disconnected  --
...
```

2. Create the `/etc/NetworkManager/conf.d/99-unmanaged-devices.conf` file with the following content:

```
 [keyfile]
 unmanaged-devices=interface-name:enp1s0
```

    To set multiple devices as unmanaged, separate the entries in the unmanaged-devices parameter with semicolon:

    ```
    [keyfile]
    unmanaged-devices=interface-name:interface_1;interface-name:interface_2;...
    ```

3. Reload the NetworkManager service:

`# systemctl reload NetworkManager`

**Verification steps**

- Display the list of devices:

```
# nmcli device status
DEVICE  TYPE      STATE      CONNECTION
enp1s0  ethernet  unmanaged  --
...
```

    The unmanaged state next to the enp1s0 device indicates that NetworkManager does not manage this device.

**Additional resources**

- For a list of criteria you can use to configure devices as unmanaged and the corresponding syntax, see the `Device List Format` section in the `NetworkManager.conf(5)` man page.

------

4.2. Temporarily configuring a device as unmanaged in NetworkManager

 You can configure devices as unmanaged based on several criteria, such as the interface name, MAC address, or device type. This procedure describes how to temporarily set the enp1s0 interface as unmanaged in NetworkManager.

Use this method, for example, for testing purposes. To permanently configure network devices as unmanaged, see Section 4.1, “Permanently configuring a device as unmanaged in NetworkManager”.

Use this method, for example, for testing purposes. To permanently configure network devices as unmanaged, see the Permanently configuring a device as unmanaged in NetworkManager section in the Configuring and managing networking documentation.

**Procedure**

1. Optional: Display the list of devices to identify the device you want to set as `unmanaged`:

```
# nmcli device status
DEVICE  TYPE      STATE         CONNECTION
enp1s0  ethernet  disconnected  --
...
```

2. Set the enp1s0 device to the unmanaged state:

`# nmcli device set enp1s0 managed no`

**Verification steps**

- Display the list of devices:

```
# nmcli device status
DEVICE  TYPE      STATE      CONNECTION
enp1s0  ethernet  unmanaged  --
...
```

    The unmanaged state next to the `enp1s0` device indicates that NetworkManager does not manage this device.

**Additional resources**

- For a list of criteria you can use to configure devices as unmanaged and the corresponding syntax, see the `Device List Format` section in the `NetworkManager.conf(5)` man page.

------

### Chapter 5. Getting started with nmtui

1. Display the status of the devices and connections:

```
# nmcli device status
DEVICE      TYPE      STATE      CONNECTION
enp1s0      ethernet  connected  Example-Connection
```

2. To display all settings of the connection profile:

```
# nmcli connection show Example-Connection
connection.id:              Example-Connection
connection.uuid:            b6cdfa1c-e4ad-46e5-af8b-a75f06b79f76
connection.stable-id:       --
connection.type:            802-3-ethernet
connection.interface-name:  enp1s0
...
```
If the configuration on the disk does not match the configuration on the device, starting or restarting NetworkManager creates an in-memory connection that reflects the configuration of the device. For further details and how to avoid this problem, see NetworkManager duplicates a connection after restart of NetworkManager service. 

------

## Chapter 6. Getting started with nmcli

This section describes general information about the `nmcli` utility.

### 6.1. The different output formats of nmcli

The `nmcli` utility supports different options to modify the output of `nmcli` commands. Using these options, you can display only the required information. This simplifies processing the output in scripts.

By default, the `nmcli` utility displays its output in a table-like format:

```
# nmcli device
DEVICE  TYPE      STATE      CONNECTION
enp1s0  ethernet  connected  enp1s0
lo      loopback  unmanaged  --
```

Using the `-f` option, you can display specific columns in a custom order. For example, to display only the `DEVICE` and `STATE` column, enter:

```
# nmcli -f DEVICE,STATE device
DEVICE  STATE
enp1s0  connected
lo      unmanaged
```

The `-t` option enables you to display the individual fields of the output in a colon-separated format:

```
# nmcli -t device
enp1s0:ethernet:connected:enp1s0
lo:loopback:unmanaged:
```

Combining the `-f` and `-t` to display only specific fields in colon-separated format can be helpful when you process the output in scripts:

```
# nmcli -f DEVICE,STATE -t device
enp1s0:connected
lo:unmanaged
```

------

### 6.2. Using tab completion in nmcli

If the `bash-completion` package is installed on your host, the nmcli utility supports tab completion. This enables you to auto-complete option names and to identify possible options and values.

------

### 6.3. Frequent nmcli commands

The following is an overview about frequently-used nmcli commands.

- To display the list connection profiles, enter:

```
# nmcli connection show
NAME    UUID                                  TYPE      DEVICE
enp1s0  45224a39-606f-4bf7-b3dc-d088236c15ee  ethernet  enp1s0
```

- To display the settings of a specific connection profile, enter:

```
# nmcli connection show connection_name
connection.id:             enp1s0
connection.uuid:           45224a39-606f-4bf7-b3dc-d088236c15ee
connection.stable-id:      --
connection.type:           802-3-ethernet
...
```

- To modify properties of a connection, enter:

```
# nmcli connection modify connection_name property value
```

    You can modify multiple properties using a single command if you pass multiple property value combinations to the command.

- To display the list of network devices, their state, and which connection profiles use the device, enter:

```
# nmcli device
DEVICE  TYPE      STATE         CONNECTION
enp1s0  ethernet  connected     enp1s0
enp8s0  ethernet  disconnected  --
enp7s0  ethernet  unmanaged     --
...
```

- To activate a connection, enter:

```
# nmcli connection up connection_name
```

- To deactivate a connection, enter:

```
# nmcli connection down connection_name
```

------

## Chapter 7. Introduction to Nmstate

Nmstate is a declarative network manager API. The nmstate package provides the libnmstate Python library and a command-line utility, nmstatectl, to manage NetworkManager on RHEL. When you use Nmstate, you describe the expected networking state using YAML or JSON-formatted instructions.

Using Nmstate has a lot of benefits. For example, it:

- Provides a stable and extensible interface to manage RHEL network capabilities
- Supports atomic and transactional operations at the host and cluster level
- Supports partial editing of most properties and preserves existing settings that are not specified in the instructions
- Provides plug-in support to enable administrators to use their own plug-ins

**Should figure this chapter out clearly later.**

------

## Chapter 8. Configuring an Ethernet connection

This section describes different ways how to configure an Ethernet connection with static and dynamic IP addresses.

### 8.1. Configuring a static Ethernet connection using nmcli

This procedure describes adding an Ethernet connection with the following settings using the nmcli utility:

- A static IPv4 address - 192.0.2.1 with a /24 subnet mask

- A static IPv6 address - 2001:db8:1::1 with a /64 subnet mask

- An IPv4 default gateway - 192.0.2.254

- An IPv6 default gateway - 2001:db8:1::fffe

- An IPv4 DNS server - 192.0.2.200

- An IPv6 DNS server - 2001:db8:1::ffbb

- A DNS search domain - example.com

**Procedure**

1. Add a new NetworkManager connection profile for the Ethernet connection:

```
# nmcli connection add con-name Example-Connection ifname eth0 autoconnect no type ethernet
```

    The further steps modify the Example-Connection connection profile you created.

2. Set the IPv4 address:

```
# nmcli connection modify Example-Connection ipv4.addresses 192.168.122.1.8/24
```

3. Set the IPv4 connection method to manual:

```
# nmcli connection modify Example-Connection ipv4.method manual
```

4. Set the IPv4 default gateways:

```
# nmcli connection modify Example-Connection ipv4.gateway 192.168.122.1
```

5. Set the IPv4 DNS server addresses:

```
# nmcli connection modify Example-Connection ipv4.dns "8.8.8.8,144.144.144.144"
```

    To set multiple DNS servers, specify them space-separated and enclosed in quotes.

    Set the IPv4 autoconnection to yes:

    # nmcli connection modify Example-Connection autoconnection yes

    Or, you can modify it like this:

    # nmcli connection modify Example-Connection \
      ifname eth0 autoconnect no type ethernet \
      ipv4.addresses "192.168.122.1.8/24" \
      ipv4.mothed "manual" \
      ipv4.geteway "192.168.1.1" \
      ipv4.dns "8.8.8.8,144.144.144.144"

    Activate the connection profile:
    # nmcli connection up Example-Connection
    Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/13)

    Deactivate the connection profile:
    # nmcli connection down Example-Connection
    Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/13)

    Delete a connection profile:
    # nmcli connection delete Example-Connection
    Connection 'Example-Connection' (acfab2d3-218d-4cbb-8bfb-5fc9c35af203) successfully deleted.

Verification steps

    Display the status of the devices and connections:

    # nmcli device status
    DEVICE      TYPE      STATE      CONNECTION
    enp7s0      ethernet  connected  Example-Connection

    To display all settings of the connection profile:

    # nmcli connection show Example-Connection
    connection.id:              Example-Connection
    connection.uuid:            b6cdfa1c-e4ad-46e5-af8b-a75f06b79f76
    connection.stable-id:       --
    connection.type:            802-3-ethernet
    connection.interface-name:  enp7s0
    ...

     Use the host utility to verify that name resolution works. For example:
     # host client.example.com
------

### Chapter 9. Managing Wi-Fi connections

------

### Chapter 10. Configuring VLAN tagging

VLAN tags for Ethernet networks follow the IEEE 802.1Q industry standard. An 802.1Q tag consists of 32 bits (4 bytes) of data inserted into the Ethernet frame header.

Note that the VLAN must be within the range from 0 to 4094.

Procedure

    Display the network interfaces:
    # nmcli device status
    DEVICE   TYPE      STATE         CONNECTION
    enp1s0   ethernet  disconnected  enp1s0
    bridge0  bridge    connected     bridge0
    bond0    bond      connected     bond0
    ...

    Create the VLAN interface. For example, to create a VLAN interface named vlan10 that uses enp1s0 as its parent interface and that tags packets with VLAN ID 10, enter:
    # nmcli connection add type vlan con-name vlan10 ifname vlan10 vlan.parent enp1s0 vlan.id 10

    Note that the VLAN must be within the range from 0 to 4094.

    By default, the VLAN connection inherits the maximum transmission unit (MTU) from the parent interface. Optionally, set a different MTU value:
    # nmcli connection modify vlan10 802-3-ethernet.mtu 2000

    Configure the IP settings of the VLAN device. Skip this step if you want to use this VLAN device as a port of other devices.

        Configure the IPv4 settings. For example, to set a static IPv4 address, network mask, default gateway, and DNS server to the vlan10 connection, enter:
        # nmcli connection modify vlan10 ipv4.addresses '192.0.2.1/24'
        # nmcli connection modify vlan10 ipv4.gateway '192.0.2.254'
        # nmcli connection modify vlan10 ipv4.dns '192.0.2.253'
        # nmcli connection modify vlan10 ipv4.method manual

    Activate the connection:
    # nmcli connection up vlan10
------

### Chapter 11. Configuring a network bridge

A network bridge is a link-layer device which forwards traffic between networks based on a table of MAC addresses. The bridge builds the MAC addresses table by listening to network traffic and thereby learning what hosts are connected to each network.

A bridge requires a network device in each network the bridge should connect. When you configure a bridge, the bridge is called `controller` and the devices it uses `ports`.





------

### Chapter 15. Configuring IP tunnels

------

### Chapter 18. Managing the default gateway setting

Procedure

    Set the IP address of the default gateway.

    For example, to set the IPv4 address of the default gateway on the example connection to 192.168.1.1:
    $ sudo nmcli connection modify example ipv4.gateway "192.168.1.1"

    Restart the network connection for changes to take effect. For example, to restart the example connection using the command line:
    $ sudo nmcli connection up example

    Optionally, verify that the route is active.

    To display the IPv4 default gateway:
    $ ip -4 route
    default via 192.0.2.1 dev example proto static metric 100

------

### Chapter 19. Configuring static routes

The procedure in this section describes how to add a route to the 192.0.2.0/24 network that uses the gateway running on 198.51.100.1, which is reachable through the example connection.

Prerequisites

    The network is configured
    The gateway for the static route must be directly reachable on the interface.
    If the user is logged in on a physical console, user permissions are sufficient. Otherwise, the command requires root permissions.

Procedure

    Add the static route to the example connection:
    $ sudo nmcli connection modify example +ipv4.routes "192.0.2.0/24 198.51.100.1"

    To set multiple routes in one step, pass the individual routes comma-separated to the command. For example, to add a route to the 192.0.2.0/24 and 203.0.113.0/24 networks, both routed through the 198.51.100.1 gateway, enter:
    $ sudo nmcli connection modify example +ipv4.routes "192.0.2.0/24 198.51.100.1, 203.0.113.0/24 198.51.100.1"

    Optionally, verify that the routes were added correctly to the configuration:
    $ nmcli connection show example
    ...
    ipv4.routes:        { ip = 192.0.2.1/24, nh = 198.51.100.1 }
    ...

    Restart the network connection:
    $ sudo nmcli connection up example

    Warning

    Restarting the connection briefly disrupts connectivity on that interface.

    Optionally, verify that the route is active:
    $ ip route
    ...
    192.0.2.0/24 via 198.51.100.1 dev example proto static metric 100

Note: you can use `nmcli connection modify connection_name -ipv4.routes "…"` to remove a specific route.

------

### Chapter 20. Configuring policy-based routing to define alternative routes




------






------

/* Display the current profile names and the associated device names: 
# nmcli -f NAME,DEVICE,FILENAME connection show

/* Reload NetworkManager:
# nmcli connection reload

/* To load a new configuration file: 
# nmcli connection load /etc/sysconfig/network-scripts/ifcfg-connection_name

/* If you updated a connection file that has already been loaded into NetworkManager, enter:
# nmcli connection up connection_name
