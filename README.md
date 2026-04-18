# ubuntu-geoclue-gps
Some notes on getting geoclue to work with a serial (over USB) attached GPS

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




## References
https://w3.cs.jmu.edu/bernstdh/web/common/help/nmea-sentences.php</br>



