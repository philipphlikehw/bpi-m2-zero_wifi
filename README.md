# bpi-m2-zero_wifi
This page describes how to enable Wifi on a Banana Pi M2-Zero board and Buildroot

## Precondition
You need a capable linux distribution. I use ubuntu22.04 by Windows Subsytem for Linux.
Next, install needed packages to initalize you system:
```apt install -y git tar bzip2 make file gcc libncurses5-dev libncursesw5-Ydev cpio unzip rsync bc g++```

Then you are basicly ready for fetching and building buildroot as normal user (not root). I used the release 2023-08. If you want to use another reales, just adape the first lines.
```cd ~    #go to home directory, or any other
BRrelease=buildroot-2023.08-rc1
wget -c -N https://www.buildroot.org/downloads/$BRrelease.tar.gz
tar -xvf $BRrelease.tar.gz
cd $BRrelease```
