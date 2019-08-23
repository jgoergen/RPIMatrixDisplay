# RPIMatrixDisplay

## Parts
* Raspberry Pi, I'm using a Zero W but it would run even faster on a 3 or 4 ( https://www.adafruit.com/categories/176 )
* Adafruit P3 Matrix Hat ( https://learn.adafruit.com/adafruit-rgb-matrix-bonnet-for-raspberry-pi )
* A high current 5v adapter ( https://www.adafruit.com/product/1466 )

## Wiring
* Follow this guide here: https://learn.adafruit.com/adafruit-rgb-matrix-bonnet-for-raspberry-pi/pinouts

## Set up

### Install Operating System and connect
* Install Raspbian Lite to SD Card
* Make text file named ssh on the boot drive ( note, no extension ) 
* Make text file named wpa_supplicant.conf on the boot drive and add this text

```
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="AP_SSID"
    scan_ssid=1
    psk="AP_PASS"
    key_mgmt=WPA-PSK
}
```
* Boot Raspberry Pi
* Find IP address on network and SSH in default ( I reccomend using Putty: https://www.putty.org/ )
```
Login: pi
Pass: raspberry
```
* Update network host name and change password
```
sudo raspi-config
```
* Update APT
```
sudo apt update
```
### Install Bonjour / Zeroconf for easier access over the network
* On windows install Apple Bonjour if you don't already have it
( https://support.apple.com/kb/DL999?locale=en_US )
* On the Raspberry pi SSH shell
```
sudo apt-get install avahi-daemon
sudo update-rc.d avahi-daemon defaults
sudo nano /etc/avahi/services/afpd.service
```
Paste the following into that file and save (right clicking in putty will paste.)
```
<?xml version="1.0" standalone='no'?><!--*-nxml-*-->
  <!DOCTYPE service-group SYSTEM "avahi-service.dtd">
  <service-group>
      <name replace-wildcards="yes">%h</name>
      <service>
          <type>_afpovertcp._tcp</type>
          <port>548</port>
      </service>
  </service-group>
</xml>
```
```
sudo /etc/init.d/avahi-daemon restart
```
* Install Samba for ease of use over the network
```
sudo apt-get install samba samba-common-bin
```
* Update samba password
```
sudo smbpasswd -a pi
```
* Edit samba config 
```
sudo nano /etc/samba/smb.conf
```
* Paste this at the bottom
```
workgroup = WORKGROUP
wins support = yes

[source]
   comment = Share
   path = /
   browseable = Yes
   writeable = Yes
   only guest = no
   create mask = 0777
   directory mask = 0777
   public = yes
   read only = no
   force user = root
   force group = root
```

### Setup Python and Libraries
* Install Pygame dependencies.
```
sudo apt-get install python-pip libsdl1.2-dev python-dev libsdl-image1.2-dev libsdl-mixer1.2-dev libsdl-ttf2.0-dev   libsdl1.2-dev libsmpeg-dev python-numpy subversion libportmidi-dev ffmpeg libswscale-dev libavformat-dev libavcodec-dev 
```
* Install Pygame
```
sudo pip install pygame
```
* Install Matrix drivers
```
curl https://raw.githubusercontent.com/adafruit/Raspberry-Pi-Installer-Scripts/master/rgb-matrix.sh >rgb-matrix.sh
sudo bash rgb-matrix.sh
```
* Follow installation guide, if you need help refer to this guide ( https://learn.adafruit.com/adafruit-rgb-matrix-bonnet-for-raspberry-pi/driving-matrices )

### Pull down Code
* Install git
```
sudo apt install git
```
* Make directory for source code and pull it down
```
mkdir Source
cd Source
```
* Pull down code
``` 
git clone https://github.com/jgoergen/RPIMatrixDisplay
```


### Run Code



### Make one of the apps Autostart
* CD to the directory the app you want to make autostart.
* Create autostart script
```
sudo nano autostart.sh
```
* Copy this into the file.
```
#!/bin/bash

sudo python ./APP_FILE_NAME.py
```
* Make the autostart shell script executeable and make it start on boot.
```
chmod +x start.sh
sudo nano /etc/rc.local
```
* Add this right above exit 0
```
cd /home/pi/PATH_TO_APP/
bash start.sh
```
### Make Raspberry Pi an accesspoint that tunnels it's internet connection
```
curl https://raw.githubusercontent.com/lukicdarkoo/rpi-wifi/master/configure | bash -s -- -a ACCESS_POINT_NAME ACCESS_POINT_PASSWORD -c AP_SSID AP_PASS
```
* Some hacks to ensure it works
```
Sudo nano /etc/rc.local
```
* Past this this before exit 0
```
sudo iptables -t nat -A POSTROUTING -s 192.168.10.0/24 ! -d 192.168.10.0/24 -j MASQUERADE
```

# More info and notes
* Matrix driver info and notes on use: https://github.com/hzeller/rpi-rgb-led-matrix