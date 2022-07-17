# iSpindel Access Point
## Summary
An iSpindel is an IoT hydrometer used in fermenting. It floats on top of the 
liquid, using the angle it is floating at to determine the density of the 
solution, which in turn can be used to determine how a fermentation is going.

The iSpindel communicates using a short-term wifi connection, sending data at
regular intervals and sleeping in between.

This project aims to provide a simple access point for the iSpindel which can
be placed in close proximity to the fermenter, along with displaying the results
in an optional, visual e-Ink display.

Benefits are:
- Can be placed in close proximity to the fermenter:
-- This means that the connection is more reliable
-- Saves power for the iSpindel
-- Take the network to the fermenter, rather than the fermenter to the network
- Can be used in an isolated environment (e.g shed, basement, attic)
- Results can be cached or stored locally
- Live results can be viewed locally (e.g. ePaper / eInk or via a monitor )

## Hardware Used
- iSpindel 
- Raspberry Pi Zero W
- (Optional) Waveshare 2.13" Touch e-Paper Hat V2 (250x122)

![Waveshare Touch e-Paper case with Raspberry Pi Zero W](/ispindel-eink-display)

## Packages Required
### Access Point
Install hostapd, dnsmasq (for DHCP server) and screen for a persistent shell:
```
$ sudo apt-get install hostapd dnsmasq screen
```

### Waveshare e-Paper

Ensure Python is fully-fledged:
```
$ sudo apt-get install git python3-pip python3-pil python3-numpy
```

These Python packages should already be active on the Raspberry Pi:
```
sudo pip3 install RPi.GPIO spidev
```

### This Repo
Also don't forget to drop this repo on there!

## Access Point Setup

Warning: By putting in this configuration, network access will be lost unless 
the access point is fully functional. Ensure there's an alternative approach to
resolve issues (i.e. keyboard & monitor to the Raspberry Pi).

Enable the SSH server, so that it's possible to log in:

```
$ sudo raspi-config
    3. Interface Options 
        I2 SSH (Enable)
```

Install hostapd, dnsmasq (for DHCP server) and screen for a persistent shell:
```
$ sudo apt-get install hostapd dnsmasq screen
```

Disable power-saving on wifi:
```
$ sudo echo "wireless-power off" > /etc/network/interfaces.d/wireless-pm-off
```

Set up the networking for the Raspberry Pi (static ip of 192.168.0.1):
```
$ sudo vi /etc/network/interfaces.d/ispindel
allow-hotplug wlan0
iface wlan0 inet static
    address 192.168.0.1
    netmask 255.255.255.0
    network 192.168.0.0
```

Stop the DHCP client daemon from interfering with our AP, but allowing it to
work with other interfaces (e.g. eth0, wlan1):
```
$ sudo echo "denyinterfaces wlan0" >> /etc/dhcpcd.conf
```

Set up dnsmasq to act as a lightweight DHCP server on wlan0 / 192.168.0.0:
```
$ sudo vi /etc/dnsmasq.d/ispindel
interface=wlan0
dhcp-range=192.168.0.2,192.168.0.254,255.255.255.0,12h
```

Set up the HostAP to do the authentication of wifi clients:
```
$ sudo vi /etc/hostapd/hostapd.conf
interface=wlan0
ssid=ispindel
hw_mode=g
channel=1
wpa=2
wpa_pairwise=TKIP
rns_pairwise=CCMP
wpa_key_mgmt=WPA-PSK
wpa_passphrase=passwordgoeshere
```

HostAP is not enabled to work on boot - enable it:
```
$ sudo systemctl unmask hostapd
$ sudo systemctl enable hostapd
```

A reboot is probably the best option to see if everything is working:
```
$ sudo reboot
```

Otherwise, restart everything piece-by-piece:
```
$ sudo systemctl restart dhcpcd
$ sudo systemctl restart networking
$ sudo systemctl restart dnsmasq
$ sudo systemctl start hostapd
```

If all is fine, it may be worth decommissioning the following services:
- dhcpcd
- wpa_supplicant

## Waveshare ePaper Preparation

Enable SPI:
```
$ sudo raspi-config
    3. Interface Options
        I4 SPI (Enable)
```

## iSpindel HTTP Server
The iSpindel server should be run inside screen:
```
$ screen
<-- Now inside screen shell -->
$ python3 ispindel-http-server.py
```

Notes:
- Edit the python to change the listening host, port and EPaper.
- Data is logged to ispindel-json.log.
- Font used is a minimal bitmap. This can a bitmap or a Truetype font.
- Font moves around the screen
