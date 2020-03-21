
![](/images/miscellaneous/rpi_router.png)

With the advent of the disastrous COVID-19, I decided to write this down as a sort of COVID-19 survival tip.

I've had to take a work project home and set it up in my home office. The work project is an embedded system with a physical Ethernet interface for networking (i.e. no WiFi), and requires internet access. I used an old Raspberry Pi 3 that I had laying around and set it up as a router to get the embedded system onto the internet.

Here are the steps.

<!-- TEASER_END -->

## Hardware
- Raspberry Pi (I used a 3 model B)
- SD card
- PC with SD card reader/writer
- PC with Balena Etcher installed:
  - I used the portable version of [Balena Etcher for Windows](https://www.balena.io/etcher/), specifically this [release](https://github.com/balena-io/etcher/releases/download/v1.5.79/balenaEtcher-Portable-1.5.79.exe)

## Raspberry Pi Initial Setup
- Download the zip file for [Raspbian Buster with desktop](https://downloads.raspberrypi.org/raspbian_full_latest). At the time, the image had a release date of 2020-02-13.

- Unzip the downloaded file to retrieve the Raspbian `.img` file.

- Flash the SD card with the `.img` file using Balena Etcher.

- Plug the SD card into the Pi and power up.

- Answer the questions, allow update, connect to your home Wifi. Change the password to something secure.

- From a shell, "`curl google.com`" and verify internet connectivity via WiFi.
  - Troubleshooting tip: The WiFi SSID and password that was configured can be found in `/etc/wpa_supplicant/wpa_supplicant.conf`.

## Unit Under Test Setup
As shown in the diagram above, the unit under test (UUT) has an interface `eth1` with a static IP address `192.168.168.167`. We want the UUT's traffic towards the internet to be routed via the Pi's `eth0` interface.

- Execute the following in the UUT to set up the Pi as the UUT's default gateway. (Making this setting permanent across reboots is beyond the scope here as it depends on the specifics of the UUT.)
```bash
route add default gw 192.168.168.1 dev eth1
```

- Verify the result. The routing table in the UUT should show the gateway:

```bash
$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.168.1   0.0.0.0         UG    0      0        0 eth1
...
```

## Router Setup
Now let's set up the Pi as a router. Note that we will *not* set up DHCP as everything is statically configured in this use case.

- Enable IPv4 forwarding: Edit `/etc/sysctl.conf` and uncomment the line `net.ipv4.ip_forward = 1`

- Create a script called `start_router.sh` with the following content and place it in the Pi's home directory `/home/pi`:

```bash
#!/bin/bash
ip_address="192.168.168.1"
netmask="255.255.255.0"
eth="eth0"
wlan="wlan0"

# enable routing
sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"

# set the `eth0` IP address
sudo ifconfig $eth $ip_address netmask $netmask

# add a route to reach UUT and anything else on the 192.168.168.0/24 network
sudo route add -net 192.168.168.0 netmask 255.255.255.0 gw 192.168.168.1 dev eth0

# set up iptables to perform NAT
sudo iptables -t nat -A POSTROUTING -o $wlan -j MASQUERADE
sudo iptables -A FORWARD -i $wlan -o $eth -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i $eth -o $wlan -j ACCEPT
```

- Set up the `/home/pi/start_router.sh` to be executed on boot by adding `/home/pi/start_router.sh` to the line before "`exit 0`" in `/etc/rc.local`. The resulting example file is shown below with the changes in bold.

<pre>
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.

# Print the IP address
_IP=$(hostname -I) || true
if [ "$_IP" ]; then
  printf "My IP address is %s\n" "$_IP"
fi
<b>
# Set RPi as router to the internet
/home/pi/start_router.sh
</b>
exit 0
</pre>

- Reboot the Pi and verify that the UUT can access the internet through the Pi with the physical connectivity set up as shown above.


## SSH Access To The Pi From A PC
- Enable the SSH server in the Pi using `sudo raspi-config`, `Interfaces`, and selecting `Enabled` for SSH. More details [here](https://www.raspberrypi.org/documentation/remote-access/ssh/).

- Add the following to `~/.ssh/config` in your PC:

```bash
Host rpi
  User pi
  Hostname 192.168.168.1
```

- Copy your public key from your PC to the Pi for password-less access in future: `ssh-copy-id rpi` (you'll have to enter the password this one time)

- Verify SSH access from the PC: `ssh rpi`

