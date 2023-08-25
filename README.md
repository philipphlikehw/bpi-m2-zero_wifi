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

## Configure
Now, there needs be be some modification in defconfig to enable WIFI support. You can do this by editiong the .conf file, or by 'make menuconfig'. The following configurations needs to be set:
* Target packages -> Hardware -> Firmware:                                                                 BR2_PACKAGE_LINUX_FRMWARE
* Target packages -> Hardware -> Firmware->linux-firmware -> WiFi-firmware -> Braodcome BRCM bcm43xxx:     BR2_PACKAGE_LINUX_FRMWARE_BRCM_BCM43XXX
* Target packages -> Networking application->wpa_supplicant:                                               BR2_PACKAGE_WPA_SUPPLICANT

Additional drivers in ther kernel needs to activated in ther kernel dev-config. The siplest way is, to use a kernel fragment file.
* Kernel -> Linux Kernel -> Additional configuration fragment file                                         BR2_LINUX_KERNEL_CONFIG_FILES


```
make bananapi_m2_zero_defconfig
#make adaptaions in menuconfig from above
```



