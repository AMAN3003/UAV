# UAV
# Intel Edison Setup
Instructions and scripts for the installation of CUSTOM ROS ON THE EDISON

## Before you start

Before starting with the installation it's a good idea to boot the Edison straight out of the box to make sure it's working. This way we can make sure we have a functional board before proceeding and we won't be mistakenly blaming setup issues if something is wrong here.
Connect one USB cable to the cosole port and then start your terminal app (see next section for more information on this). Once you are connected plug in the second USB cable for power and after 15 seconds you should see the system booting. If you want to login the user name is root (no password).

## Flash Debian
To build Debian Jessie carefully follow the instruction from ros-setups/README.md
If Windows is used, dfu-util is required. Download the latest version from this page:
http://dfu-util.sourceforge.net/releases/
Make sure you have the console USB cable in place and use it so you know when the installation has finished.
You MUST NOT remove power before it’s done or it could be bricked.
If you don't have a console connection make sure you wait 2 minutes at the end of the installation as it instructs.
During this time it is completing the installation which should n't be interrupted. 
If you don't get any update on your console after this message is displayed restart your console terminal connection.

Connect to the console with 115000 8N1, for example:
`screen /dev/USB0 115200 8N1`
and login as username px4 (password: px4) 

## Post Debian Install
```
After Debian has been installed you will end up with the following partitions:
Filesystem       Size  Used Avail Use% Mounted on
rootfs           1.4G  813M  503M  62% /
/dev/root        1.4G  813M  503M  62% /
devtmpfs         480M     0  480M   0% /dev
tmpfs             97M  292K   96M   1% /run
tmpfs            5.0M     0  5.0M   0% /run/lock
tmpfs            193M     0  193M   0% /run/shm
tmpfs            481M     0  481M   0% /tmp
/dev/mmcblk0p7    32M  5.3M   27M  17% /boot
/dev/mmcblk0p10  1.3G  2.0M  1.3G   1% /home
```
For some reasons, the /home partition is not mounted after the first boot. To fix this issue, add the follow to the bottom of /etc/fstab .

`sudo nano /etc/fstab` 

Add the below line in the file of fstab

/dev/disk/by-partlabel/home     /home       auto    defaults     1   1

after saving the file 

`sudo resize2fs /dev/mmcblk0p5`

If the above not worked try the below as rootfs is in partion 8
mmcblk0p* here * denote the partion no 

`sudo resize2fs /dev/mmcblk0p8`

## Wifi Setup
copy the interface to home one 
Run `sudo cp /etc/network/interfaces /etc/network/interfaces.home`

Run wpa_passphrase your wifi-ssid your-wifi-password to generate pka
Like 
Run `wpa_passphrase cs2@sutd CsTwo@SuTd`
`cd /etc/network`

Edit both "/etc/network/interfaces.home" and "/etc/network/interfaces.work"

Add the following line in interaface and interface.home file
```
# interfaces(5) file used by ifup(8) and ifdown(8)
auto lo  
iface lo inet loopback

auto wlan0  
iface wlan0 inet dhcp  
    wpa-ssid ExampleWifi
    wpa-psk 81088ba3b4b387ea4d22a4ad369ffa42f4966d2f3d61f6c65cdc001460239dc4
```
- Change wpa-ssid to your ssid 
- Change wpa-pka to generated pka 
- Comment out `auto usb0` plus the three lines that follow it (interface definition)
-Uncomment `auto wlan0`
-Save 
-Run: `ifup wlan0`
Create two script one for homenet Run `sudo nano ~/homenet.sh` and 
other for the worknet
Run `sudo nano ~/worknet.sh` 
Make script executable
`sudo chmod +x ~/homenet.sh`
`sudo chmod +x ~/worknet.sh`

Fill the following in the script

<pre><code>
#!/bin/bash
# Change Network to Home Network
cp /etc/network/interfaces.home /etc/network/interfaces
echo "Disable wlan0"
ifdown wlan0
echo "Re-enable wlan0"
ifup wlan0
</code></pre>

## Update
Add the following to the bottom of sources list (/etc/apt/sources.list)
Run`sudo nano /etc/apt/sources.list`

add the following line to the sources list for updating it from other server

```
deb http://ftp.sg.debian.org/debian jessie main contrib non-free
#deb-src http://http.debian.net/debian jessie main contrib non-free

deb http://ftp.sg.debian.org/debian jessie-updates main contrib non-free
#deb-src http://http.debian.net/debian jessie-updates main contrib non-free

deb http://security.debian.org/ jessie/updates main contrib non-free
#deb-src http://security.debian.org/ jessie/updates main contrib non-free

#deb http://ubilinux.org/edison wheezy main

deb http://ftp.sg.debian.org/debian jessie-backports main
```

execute the below line after saving the file 

Run `sudo apt-get -y update`

Run `sudo apt-get -f install`
Run `sudo apt-get -y upgrade`

