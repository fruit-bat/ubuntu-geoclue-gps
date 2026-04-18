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

Check the gnss-share socket is working
```
nc -U /var/run/gnss-share.sock
```

