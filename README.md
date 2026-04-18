# ubuntu-geoclue-gps
Some notes on getting geoclue to work with a serial (over USB) attached GPS

This is a guide mostly to remind myself how to get Untuntu desktop working with a GPSs module.
The big takeaway, is *not* to install gpsd! The desktop uses geoclue, which does not like talking to gpsd.

The magic combination, is gnss-share + geoclue-2.0 + some mapping software

I have tried OrganicMaps, which now updates my current position (yay!). 

## Setup geoclue and gnss-share
Install geoclue and gnss-share:
```sh
sudo apt-get install geoclue-2.0 geoclue-2.0-demo gnss-share
```

Edit the gnss-share config to use the serial attached GPS, in this case it is on ttyACM0.

```sh
sudo vi /etc/gnss-share.conf
```

```sh
# Socket to sent NMEA location to
socket="/var/run/gnss-share.sock"
# Group to set as owner for the socket
group="geoclue"

# GPS device driver to use
# Supported values: stm, stm_serial
#device_driver="stm"
device_driver="stm_serial"

# Path to GPS device to use
#device_path="/dev/gnss0"
device_path="/dev/ttyACM0"

# Baud rate for GPS serial device
#device_baud_rate=9600
device_baud_rate=115200

# Directory to load/store almanac and ephemeris data
agps_directory="/var/cache/gnss-share"
```

Restart the gnss-share service:
```sh
sudo systemctl restart gnss-share
```
Check the socket permissions:
```sh
ls -l /var/run/gnss-share.sock
srw-rw---- 1 root geoclue 0 Apr 18 15:24 /var/run/gnss-share.sock
```

Check the gnss-share socket is working
```sh
sudo nc -U /var/run/gnss-share.sock
```
You should see some NMEA output e.g.
```sh
$GPGSV,2,1,08,02,74,042,45,04,18,190,36,07,67,279,42,12,29,323,36*77
$GPGSV,2,2,08,15,30,050,47,19,09,158,,26,12,281,40,27,38,173,41*7B
```
Edit the geoclue configuration:
```sh
sudo vi /etc/geoclue/geoclue.conf
```

Change the NMEA section to use the socket provided by gnss-share:
```sh
# Network NMEA source configuration options
[network-nmea]

# Fetch location from NMEA sources on local network?
enable=true

# use aa nmea unix socket as the data source
# nnmea-socket=
nmea-socket=/var/run/gnss-share.sock
```
While you are at it, disable pretty much every other source.

Restart the geoclue service:
```sh
sudo systemctl restart geoclue
```
Check how it is getting on...
```sh
sudo journalctl -f -u geoclue
```

Check location data is being presented:
```sh
/usr/libexec/geoclue-2.0/demos/where-am-i
```

You should get something like:
```sh
New location:
Latitude:    69.841348°
Longitude:   -4.046143°
Accuracy:    0.000000 meters
Altitude:    96.300000 meters
Speed:       0.252076 meters/second
Heading:     135.000000°
Description: GPS GGA
Timestamp:   Sat 18 Apr 2026 03:42:17 PM BST (1776523337 seconds since the Epoch)
```

## Trouble shooting
If geoclue is unable to read from the gnss-share socket, adding permissions to the service may help:
```sh
sudo systemctl edit geoclue.service
```
And add the following lines in the area prompted:
```sh
[Service]
ReadWritePaths=/var/run/gnss-share.sock
# Alternatively, if /var/run is a symlink:
# ReadWritePaths=/run/gnss-share.sock
```

## Install OrganicMaps
```sh
flatpak install flathub app.organicmaps.desktop
flatpak run app.organicmaps.desktop
```

## References
https://w3.cs.jmu.edu/bernstdh/web/common/help/nmea-sentences.php</br>



