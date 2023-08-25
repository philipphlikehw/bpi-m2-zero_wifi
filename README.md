# bpi-m2-zero_wifi
This page describes how to enable Wifi on a Banana Pi M2-Zero board and Buildroot

## Precondition
You need a capable linux distribution. I use ubuntu22.04 by Windows Subsytem for Linux.
Next, install needed packages to initalize you system:
```
apt install -y git tar bzip2 make file gcc libncurses5-dev libncursesw5-Ydev cpio unzip rsync bc g++
#more tools can be neede by using other linux system
```

## Fetching
Then you are basicly ready for fetching and building buildroot as normal user (not root). I used the release 2023-08. If you want to use another reales, just adape the first lines.
```
cd ~    #go to home directory, or any other
BRrelease=buildroot-2023.08-rc1
wget -c -N https://www.buildroot.org/downloads/$BRrelease.tar.gz
tar -xvf $BRrelease.tar.gz
cd $BRrelease
```

## Configure and building
Now, there needs be be some modification in defconfig to enable WIFI support. You can do this by editiong the .conf file, or by 'make menuconfig'. The following configurations needs to be set:
* Target packages -> Hardware -> Firmware:                                                                 BR2_PACKAGE_LINUX_FRMWARE
* Target packages -> Hardware -> Firmware->linux-firmware -> WiFi-firmware -> Braodcome BRCM bcm43xxx:     BR2_PACKAGE_LINUX_FRMWARE_BRCM_BCM43XXX
* Target packages -> Networking application->wpa_supplicant:                                               BR2_PACKAGE_WPA_SUPPLICANT

Additional drivers in ther kernel needs to activated in ther kernel dev-config. The siplest way is, to use a kernel fragment file.
* Kernel -> Linux Kernel -> Additional configuration fragment file                                         BR2_LINUX_KERNEL_CONFIG_FILES
to the value `custom/kernel-config-frag`


```
make bananapi_m2_zero_defconfig
#make adaptaions in menuconfig from above
mkdir custom
touch custom/kernel-config-frag
echo echo 'CONFIG_DEBUG_FS=y'>>custum/kernel-config-frag
echo echo 'CONFIG_NET=y'>>custum/kernel-config-frag
echo echo 'CONFIG_FW_LOADER=y'>>custum/kernel-config-frag
echo echo 'CONFIG_CRYPTO_SHA256=y'>>custum/kernel-config-frag
echo echo 'CONFIG_BRCMFMAC=m'>>custum/kernel-config-frag
echo echo 'CONFIG_BRCMFMAC_SDIO=y'>>custum/kernel-config-frag
echo echo 'CONFIG_NETDEVICES=y'>>custum/kernel-config-frag
echo echo 'CONFIG_ETHERNET=y'>>custum/kernel-config-frag
echo echo 'CONFIG_NET_VENDOR_ALACRITECH=y'>>custum/kernel-config-frag
echo echo 'CONFIG_NET_VENDOR_ALLWINNER=y'>>custum/kernel-config-frag
echo echo 'CONFIG_WLAN=y'>>custum/kernel-config-frag
echo echo 'CONFIG_WLAN_VENDOR_BROADCOM=y'>>custum/kernel-config-frag
make
```
Alternativly, you can just clone this git
```
git clone https://github.com/philipphlikehw/bpi-m2-zero_wifi.git
mv bpi-m2-zero_wifi/* .
rm bpi-m2-zero_wifi
make
```

## Preparing HW
After you start build process, you have some time to prepare HW. Therefore put a WIFI-Antenna on the WiFI-Antenna-Socket. I use the following part:[ebay link](https://www.ebay.de/itm/305052000140?hash=item4706841f8c:g:xX0AAOSwM-hdSpvg&amdata=enc%3AAQAIAAAA4Ec%2FoE4B3RXrgIDFDxCZV5NDmSckyCaFuZGAIDFQqk%2BLtvF%2BVzHLoP0conzRzSrv9GNmoWvbqaPizwiJHNe5%2BYlx3PBCmkyXMzf3xtmul%2Bt0Uao%2FRkQMDM%2BKWd2orluRx2yzmqNYTBVb3Ks5odGbMOZu6xYk5aJGSJx3cPFPMENZioKql4yoynxfEh6q7HhnqGS%2BfMOeZmX%2BTVE0eVb8QGcQUzDWa0jAOezYSZsqsfWr9jc2xzgJfK%2FlneySI%2BbYiH8QM1LquLpSrNWY0lQsZz4qg2BWjBDYitRaoXqloTeF%7Ctkp%3ABk9SR4rkwOfFYg)
Then add a UART-Adapter to the Pin-Socket and connect it with the PCB. The default bautrate is adapted to 115200 baut.

![WH board setup](IMG_20230825_225620.jpg)


## Prepare SD-Card
After building is compleate suseccfull, the sd-card image is located in 'output/images/sdcard.img'. Write this image to a sdcard, using convential tools. For example 'Win 32 Disk Imager'
Plug the written SD-Card into the board and power the board by USB.
To default TTY is configured to the UART on the Headers. So connect a UART-Adapter and start the communication with 115200 baut.

## Booting and Login'
Some seconds later, a login promt shall be visible. Login-user is 'root'. A password is not needed.

## Load Driver
Load the driver by 'modprobe' command. The loading can be automated to put it init init-script. 
```
# modprobe brcmfmac
# [   67.333595] brcmfmac: brcmf_fw_alloc_request: using brcm/brcmfmac43430-sdio for chip BCM43430/1
[   67.513865] brcmfmac: brcmf_fw_alloc_request: using brcm/brcmfmac43430-sdio for chip BCM43430/1
[   67.530610] brcmfmac: brcmf_c_preinit_dcmds: Firmware: BCM43430/1 wl0: Mar 30 2021 01:12:21 version 7.45.98.118 (7d96287 CY) FWID 01-32059766
```

## Connect to WiFi Acces Point
In the last step, you yust have to create a fille witth your WiFi-Access and set an ip. Then you are 
```
# yourSSID=""
# yourpsk=""
# rm /etc/wpa_supplicant.conf
# touch /etc/wpa_supplicant.conf
# echo 'update_config=1'>>/etc/wpa_supplicant.conf
# echo 'network={'>>/etc/wpa_supplicant.conf
# echo 'ssid="'$yourSSID'"'>>/etc/wpa_supplicant.conf
# echo 'psk="'$yourpsk'"'>>/etc/wpa_supplicant.conf
# echo 'key_mgmt=WPA-PSK'>>/etc/wpa_supplicant.conf
# echo '}'>>/etc/wpa_supplicant.conf

# wpa_supplicant -i wlan0 -c /etc/wpa_supplicant.conf &
# Successfully initialized wpa_supplicant
rfkill: Cannot open RFKILL control device
wlan0: Trying to associate with SSID '....
wlan0: Associated with 38:10:d5:ce:50:53
wlan0: CTRL-EVENT-CONNECTED - Connection to 38:10:d5:ce:50:53 completed [id=0 id_str=]
wlan0: CTRL-EVENT-SUBNET-STATUS-UPDATE status=0

# ifconfig wlan0 192.168.178.111 up
# ping 192.168.178.1
PING 192.168.178.1 (192.168.178.1): 56 data bytes
64 bytes from 192.168.178.1: seq=0 ttl=64 time=19.045 ms
64 bytes from 192.168.178.1: seq=1 ttl=64 time=4.680 ms
64 bytes from 192.168.178.1: seq=2 ttl=64 time=4.684 ms
64 bytes from 192.168.178.1: seq=3 ttl=64 time=5.337 ms

```

## Auto Connect
Once you created the file '/etc/wpa_supplicant.conf' and tested our wifi-connection, you can automate the driver-loading, connecting, and ip-assigning by a simply entry in the file '/etc/network/interfaces'
Yust add the following content into the file:
```
auto wlan0
iface wlan0 inet dhcp
        hostname BPI
        pre-up modprobe brcmfmac && while [ ! -e /sys/class/net/wlan0 ]; do sleep 1 && echo " ."; done && wpa_supplicant -D nl80211 -i wlan0 -c /etc/wpa_supplicant.conf -B
        post-down killall -q wpa_supplicant
        *wait-delay 15*

#
```

## Limitation
The BPI Board does not detect a WiFi-Access-Poitn at every Channel. If you have the issue with that you ssid is not detected, disable auto-channel in your router and set is to Channel 1.






