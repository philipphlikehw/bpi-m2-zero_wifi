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







