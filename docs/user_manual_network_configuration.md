
## network configuration

The network configuration is found in the service script `/etc/rc.d/net`. By default this service script is not configured to connect to the network.
Example:

```bash
#!/bin/bash
#
# /etc/rc.d/net: start/stop network interface
#

DHCP_OPTS="-t 10 -b"

# If USE_DHCP[interface] is set to "yes", this overrides any other settings.
# If you don't have an interface, leave the settings null ("").

# ----------------------------------------

# ipv4 config options for eth0
DEV[0]=""
ADDR[0]=""
MASK[0]=""
USE_DHCP[0]=""

# ipv4 config options for eth1
DEV[1]=""
ADDR[1]=""
MASK[1]=""
USE_DHCP[1]=""

# ipv4 config options for eth2
DEV[2]=""
ADDR[2]=""
MASK[2]=""
USE_DHCP[2]=""

# ipv4 default gateway ip address
GW=""

# ----------------------------------------

case $1 in
	start)
		for i in ${!DEV[@]}; do
			if [ "${USE_DHCP[$i]}" = "yes" ]; then
				/sbin/dhcpcd ${DHCP_OPTS} ${DEV[$i]}
			elif [ ! -z "${DEV[$i]}" -a ! -z "${ADDR[$i]}" -a ! -z "${MASK[$i]}" ]; then
				/sbin/ip addr add ${ADDR[$i]}/${MASK[$i]} dev ${DEV[$i]} broadcast +
				/sbin/ip link set ${DEV[$i]} up
			fi
		done
		if [ ! -z $GW ] ; then
			/sbin/ip route add default via ${GW}
		fi
		;;
	stop)
		if [ ! -z $GW ] ; then
			echo /sbin/ip route del default
		fi
		for i in ${!DEV[@]}; do
			if [ "${USE_DHCP[$i]}" = "yes" ]; then
				/sbin/dhcpcd -k ${DEV[$i]}
			elif [ ! -z "${DEV[$i]}" -a ! -z "${ADDR[$i]}" -a ! -z "${MASK[$i]}" ]; then
				/sbin/ip link set ${DEV[$i]} down
				/sbin/ip addr del ${ADDR[$i]}/${MASK[$i]} dev ${DEV[$i]}
			fi
		done
		;;
	restart)
		$0 stop
		sleep 1
		$0 start
		;;
	*)
		echo "Usage: $0 [start|stop|restart]"
		;;
esac

# End of file
```

To enable a network with dynamic address acquisition (dhcp), you must specify the network interface and enable the use of dhcp:

```bash
DEV[0]="eth0"
ADDR[0]=""
MASK[0]=""
USE_DHCP[0]="yes"
```

For WiFi, you must make the following settings `/etc/rc.d/net`:

```bash
DEV[0]="wlan0"
ADDR[0]=""
MASK[0]=""
USE_DHCP[0]="yes"
```
And also configure `/etc/rc.d/wpa_supplicant`:

```bash
DEV=wlan0
```
And configure authorization in `/etc/wpa_supplicant.conf`:

```bash
network={
   ssid="register the name of the access point"
   scan_ssid=1
   key_mgmt=WPA-PSK
   psk="register a password"
}
```

If you need to work with a static address, make the following settings:
```bash
DEV[0]="eth0"
ADDR[0]="192.168.0.254"
MASK[0]="255.255.255.0"
USE_DHCP[0]=""

# ipv4 default gateway ip address
GW="192.168.0.1"
```

You will also need to configure DNS settings in `/etc/resolv.conf`:
```bash
#
# /etc/resolv.conf: resolver configuration file
#

search your internal domain>
nameserver your DNS server>

# End of file
```
The wpa_supplicant package provides two startup scripts in `/etc/rc.d`. You might choose to put `wlan` in the SERVICES array of `/etc/rc.conf`, which will let **wpa_supplicant** manage all your network interfaces. Another option is to let the net startup script call wpa_supplicant as needed, by copying into `/lib/dhcpcd/dhcpcd-hooks/` the example file `/usr/share/dhcpcd/hooks/10-wpa_supplicant`.

