
<figure markdown="span">
<div id="ascii-art">
   _                                  _         _    _           
 _| | ___  ___  _ _  _____  ___  ___ | |_  ___ | |_ |_| ___  ___ 
| . || . ||  _|| | ||     || -_||   ||  _|| .'||  _|| || . ||   |
|___||___||___||___||_|_|_||___||_|_||_|  |__,||_|  |_||___||_|_|

</div>
</figure>


## network configuration


### network adapters

The network configuration is found in the service script `/etc/rc.d/net`. By default this service script is not configured to connect to the network.  
Example:
```bash
# ----------------------------------------

# ipv4 config options for eth0, eth0.10 or bond0
IFNAME[0]=""
IFADDRS[0]=""
USE_DHCP[0]=""
DHCP_HOSTNAME[0]=""

# ipv4 config options for eth1, eth1.10 or bond1
IFNAME[1]=""
IFADDRS[1]=""
USE_DHCP[1]=""
DHCP_HOSTNAME[1]=""

# ipv4 config options for eth2, eth2.10 or bond2
IFNAME[2]=""
IFADDRS[2]=""
USE_DHCP[2]=""
DHCP_HOSTNAME[2]=""

# ipv4 config options for eth3, eth3.10 or bond3
IFNAME[3]=""
IFADDRS[3]=""
USE_DHCP[3]=""
DHCP_HOSTNAME[3]=""

# ipv4 default gateway ip address
GW=""

# ----------------------------------------
```

To enable a network with dynamic address acquisition (dhcp), you must specify the network interface and enable the use of dhcp:
```bash
IFNAME[0]="eth0"
IFADDRS[0]=""
USE_DHCP[0]="yes"
DHCP_HOSTNAME[0]=""
```

To set up a static address, perform the following settings:
```bash
IFNAME[0]="eth0"
IFADDRS[0]="192.168.0.1/24"
USE_DHCP[0]=""
DHCP_HOSTNAME[0]=""

# ipv4 default gateway ip address
GW="192.168.0.1"
```

### wifi network adapters

To configure Wi-Fi adapters, you must configure the following settings in the file `/etc/rc.d/wlan`:
```bash
# ----------------------------------------

# ipv4 config options for wlan0
IFNAME[0]=""
IFADDRS[0]=""
USE_DHCP[0]=""
DHCP_HOSTNAME[0]=""

# ipv4 config options for wlan1
IFNAME[1]=""
IFADDRS[1]=""
USE_DHCP[1]=""
DHCP_HOSTNAME[1]=""

# ipv4 default gateway ip address
GW=""

# ----------------------------------------
```

To enable a network with dynamic address acquisition (dhcp), you must specify the network interface and enable the use of dhcp:
```bash
IFNAME[0]="wlan0"
IFADDRS[0]=""
USE_DHCP[0]="yes"
DHCP_HOSTNAME[0]=""
```

And also configure `/etc/rc.d/wpa_supplicant`, add or change adapter names, space separator:
```bash
IFNAME=(wlan0 wlp3s0)
```

You also need to set up authorization in `/etc/wpa_supplicant.conf`:
```bash
network={
   ssid="register the name of the access point"
   scan_ssid=1
   key_mgmt=WPA-PSK
   psk="register a password"
}
```

The wpa_supplicant package provides two startup scripts in `/etc/rc.d`.  
You might choose to put `wlan` in the SERVICES array of `/etc/rc.conf`, which will let `wpa_supplicant` manage all your network interfaces.  
Another option is to let the net startup script call wpa_supplicant as needed, by copying into `/lib/dhcpcd/dhcpcd-hooks/` the example file `/usr/share/dhcpcd/hooks/10-wpa_supplicant`.

### dns settings

You will also need to configure DNS settings in `/etc/resolv.conf`:
```bash
#
# /etc/resolv.conf: resolver configuration file
#

search your internal domain>
nameserver your DNS server>

# End of file
```

### vlan network adapters

To configure VLAN interfaces, you must install the `vlan` package.  
The VLAN ID is taken from the full interface name, which is comprised of the nderlying interface name, a period (.) and then the VLAN ID.  
IFOPTS is a pipe (|) delimited list of VLAN module specific settings to be applied to the interface.  
See the ip-link(8) man page (search for "VLAN Type Support") for details of the options available.  
This option is not required for a standard VLAN to be configured.
```bash
# ----------------------------------------

# config options for eth0.10
IFNAME[0]="eth0.10"
IFOPTS[0]=""

# config options for eth1.10
IFNAME[1]=""
IFOPTS[1]=""

# config options for eth2.10
IFNAME[2]=""
IFOPTS[2]=""

# config options for eth3.10
IFNAME[3]=""
IFOPTS[3]=""

# ----------------------------------------
```

After configuring interfaces, they must be specified in `/etc/rc.d/net` so that these interfaces can receive an address.  
To configure this service `vlan` at system startup, add it before the `net` in the SERVICES array of `/etc/rc.conf`.

### bonding network adapters

To configure bond (link aggregation) interfaces, you must install the `bonding` package.  
Parameter description:  
- BONDNICS is a space delimited list of interfaces to add to this bond.  
- BONDMODE sets the bonding mode for this interface.  If not specified when has been used, the default is 'balance-rr'.  
- IFOPTS is a pipe (|) delimited list of bonding module specific settings to be applied to the interface, and should always include the 'miimon' option when configuring bonding - not using this option will result in network degradation. In 'active-backup' mode, the 'primary' option should also be supplied. When using '802.3ad' mode, set "lacp_rate fast" for faster recovery from an interface failure. In other modes, the 'xmit_hash_policy' should be set.  
See the https://www.kernel.org/doc/Documentation/networking/bonding.txt.  

```bash
# ----------------------------------------

# ipv4 config options for bond0
IFNAME[0]="bond0"
BONDNICS[0]="eth0 eth1"
BONDMODE[0]="balance-rr"
IFOPTS[0]="xmit_hash_policy layer2+3 | miimon 100"

# ipv4 config options for bond1
IFNAME[1]=""
BONDNICS[1]=""
BONDMODE[1]=""
IFOPTS[1]=""

# ----------------------------------------

```

After configuring interfaces, they must be specified in `/etc/rc.d/net` so that these interfaces can receive an address.  
To configure this service `bonding` at system startup, add it before the `vlan` or `net` in the SERVICES array of `/etc/rc.conf`.

