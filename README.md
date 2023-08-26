# bpi-m2-zero_wifi
This guide outlines the steps to enable Wi-Fi on a Banana Pi M2-Zero board using Buildroot.

## Precondition
You'll need a capable Linux distribution. I'm using Ubuntu 22.04 through Windows Subsystem for Linux. Install the necessary packages to prepare your system:
```console
sudo apt install -y git tar bzip2 make file gcc libncurses5-dev libncursesw5-Ydev cpio unzip rsync bc g++
# Additional tools might be required depending on your Linux distribution
```

## Fetching
Once your system is prepared, you can fetch and build Buildroot as a regular user (not as root). I'm using version 2023-08. If you wish to use a different release, adjust the initial lines accordingly:
```console
cd ~    # Navigate to the home directory or any other directory of your choice
BRrelease=buildroot-2023.08-rc1
wget -c -N https://www.buildroot.org/downloads/$BRrelease.tar.gz
tar -xvf $BRrelease.tar.gz
cd $BRrelease
```

## Configuration and Compilation
To enable Wi-Fi support, some modifications are needed in the configuration. You can do this either by editing the .config file directly or using the make menuconfig command. The following configurations need to be set:

* Target packages -> Hardware -> Firmware: BR2_PACKAGE_LINUX_FIRMWARE
* Target packages -> Hardware -> Firmware->linux-firmware -> WiFi-firmware -> Broadcom BRCM bcm43xxx: BR2_PACKAGE_LINUX_FIRMWARE_BRCM_BCM43XXX
* Target packages -> Networking applications -> wpa_supplicant: BR2_PACKAGE_WPA_SUPPLICANT
Additional drivers in the kernel need to be activated in the kernel dev configuration. The simplest way is to use a kernel fragment file:

* Kernel -> Linux Kernel -> Additional configuration fragment file BR2_LINUX_KERNEL_CONFIG_FILES
set to the value 'custom/kernel-config-frag'


```console
make bananapi_m2_zero_defconfig
# Make adaptations in menuconfig as described above
mkdir custom
touch custom/kernel-config-frag
echo 'CONFIG_DEBUG_FS=y'>>custom/kernel-config-frag
echo 'CONFIG_NET=y'>>custom/kernel-config-frag
echo 'CONFIG_FW_LOADER=y'>>custom/kernel-config-frag
echo 'CONFIG_CRYPTO_SHA256=y'>>custom/kernel-config-frag
echo 'CONFIG_BRCMFMAC=m'>>custom/kernel-config-frag
echo 'CONFIG_BRCMFMAC_SDIO=y'>>custom/kernel-config-frag
echo 'CONFIG_NETDEVICES=y'>>custom/kernel-config-frag
echo 'CONFIG_ETHERNET=y'>>custom/kernel-config-frag
echo 'CONFIG_NET_VENDOR_ALACRITECH=y'>>custom/kernel-config-frag
echo 'CONFIG_NET_VENDOR_ALLWINNER=y'>>custom/kernel-config-frag
echo 'CONFIG_WLAN=y'>>custom/kernel-config-frag
echo 'CONFIG_WLAN_VENDOR_BROADCOM=y'>>custom/kernel-config-frag
make
```
Alternatively, you can clone this Git repository:
```console
git clone https://github.com/philipphlikehw/bpi-m2-zero_wifi.git
mv bpi-m2-zero_wifi/* .
rm -r bpi-m2-zero_wifi
make
```

## Preparing HW
While the compilation is in progress, you have time to prepare the hardware. Attach a Wi-Fi antenna to the corresponding socket. I'm using the following part: [ebay link](https://www.ebay.de/itm/305052000140?hash=item4706841f8c:g:xX0AAOSwM-hdSpvg&amdata=enc%3AAQAIAAAA4Ec%2FoE4B3RXrgIDFDxCZV5NDmSckyCaFuZGAIDFQqk%2BLtvF%2BVzHLoP0conzRzSrv9GNmoWvbqaPizwiJHNe5%2BYlx3PBCmkyXMzf3xtmul%2Bt0Uao%2FRkQMDM%2BKWd2orluRx2yzmqNYTBVb3Ks5odGbMOZu6xYk5aJGSJx3cPFPMENZioKql4yoynxfEh6q7HhnqGS%2BfMOeZmX%2BTVE0eVb8QGcQUzDWa0jAOezYSZsqsfWr9jc2xzgJfK%2FlneySI%2BbYiH8QM1LquLpSrNWY0lQsZz4qg2BWjBDYitRaoXqloTeF%7Ctkp%3ABk9SR4rkwOfFYg)
Additionally, connect a UART adapter to the pin socket and connect it to the PCB. The default baud rate is set to 115200.

![WH board setup](IMG_20230825_225620.jpg)


## Prepare SD-Card
Once the build process is successfully completed, the SD card image is located in 'output/images/sdcard.img'. Write this image to an SD card using conventional tools, such as 'Win32 Disk Imager'. Insert the written SD card into the board and power the board using USB. The default TTY is configured to use the UART on the headers. Therefore, connect a UART adapter and start communication at a baud rate of 115200.

## Booting and Login
After a few seconds, a login prompt should appear. The login user is 'root', and no password is required.
```console
U-Boot SPL 2020.10 (Aug 25 2023 - 14:13:30 +0200)
DRAM: 512 MiB
Trying to boot from MMC1


U-Boot 2020.10 (Aug 25 2023 - 14:13:30 +0200) Allwinner Technology

CPU:   Allwinner H3 (SUN8I 1680)
Model: Banana Pi BPI-M2-Zero
DRAM:  512 MiB
MMC:   mmc@1c0f000: 0, mmc@1c10000: 1
Loading Environment from FAT... *** Warning - bad CRC, using default environment

In:    serial
Out:   serial
Err:   serial
Net:   No ethernet found.
starting USB...
No working controllers found
Hit any key to stop autoboot:  0
switch to partitions #0, OK
mmc0 is current device
Scanning mmc 0:1...
Found U-Boot script /boot.scr
294 bytes read in 1 ms (287.1 KiB/s)
## Executing script at 43100000
switch to partitions #0, OK
mmc0 is current device
4882504 bytes read in 223 ms (20.9 MiB/s)
22739 bytes read in 3 ms (7.2 MiB/s)
## Flattened Device Tree blob at 43000000
   Booting using the fdt blob at 0x43000000
   Loading Device Tree to 49ff7000, end 49fff8d2 ... OK

Starting kernel ...

[    0.000000] Booting Linux on physical CPU 0x0
[    0.000000] Linux version 5.9.11 (philipp@LAPTOP-PNCFPR4S) (arm-buildroot-linux-gnueabihf-gcc.br_real (Buildroot 2023.08-rc1) 12.3.0, GNU ld (GNU Binutils) 2.40) #1 SMP Fri Aug 25 14:13:51 CEST 2023
[    0.000000] CPU: ARMv7 Processor [410fc075] revision 5 (ARMv7), cr=10c5387d
[    0.000000] CPU: div instructions available: patching division code
[    0.000000] CPU: PIPT / VIPT nonaliasing data cache, VIPT aliasing instruction cache
[    0.000000] OF: fdt: Machine model: Banana Pi BPI-M2-Zero
[    0.000000] Memory policy: Data cache writealloc
[    0.000000] cma: Reserved 16 MiB at 0x5f000000
[    0.000000] Zone ranges:
[    0.000000]   Normal   [mem 0x0000000040000000-0x000000005fffffff]
[    0.000000]   HighMem  empty
[    0.000000] Movable zone start for each node
[    0.000000] Early memory node ranges
[    0.000000]   node   0: [mem 0x0000000040000000-0x000000005fffffff]
[    0.000000] Initmem setup node 0 [mem 0x0000000040000000-0x000000005fffffff]
[    0.000000] psci: probing for conduit method from DT.
[    0.000000] psci: Using PSCI v0.1 Function IDs from DT
[    0.000000] percpu: Embedded 15 pages/cpu s30924 r8192 d22324 u61440
[    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 130048
[    0.000000] Kernel command line: console=ttyS0,115200 earlyprintk root=/dev/mmcblk0p2 rootwait
[    0.000000] Dentry cache hash table entries: 65536 (order: 6, 262144 bytes, linear)
[    0.000000] Inode-cache hash table entries: 32768 (order: 5, 131072 bytes, linear)
[    0.000000] mem auto-init: stack:off, heap alloc:off, heap free:off
[    0.000000] Memory: 490988K/524288K available (7168K kernel code, 507K rwdata, 2040K rodata, 1024K init, 248K bss, 16916K reserved, 16384K cma-reserved, 0K highmem)
[    0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=4, Nodes=1
[    0.000000] rcu: Hierarchical RCU implementation.
[    0.000000] rcu:     RCU restricting CPUs from NR_CPUS=8 to nr_cpu_ids=4.
[    0.000000] rcu: RCU calculated value of scheduler-enlistment delay is 10 jiffies.
[    0.000000] rcu: Adjusting geometry for rcu_fanout_leaf=16, nr_cpu_ids=4
[    0.000000] NR_IRQS: 16, nr_irqs: 16, preallocated irqs: 16
[    0.000000] GIC: Using split EOI/Deactivate mode
[    0.000000] random: get_random_bytes called from start_kernel+0x320/0x4cc with crng_init=0
[    0.000000] clocksource: timer: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 79635851949 ns
[    0.000000] arch_timer: cp15 timer(s) running at 24.00MHz (phys).
[    0.000000] clocksource: arch_sys_counter: mask: 0xffffffffffffff max_cycles: 0x588fe9dc0, max_idle_ns: 440795202592 ns
[    0.000007] sched_clock: 56 bits at 24MHz, resolution 41ns, wraps every 4398046511097ns
[    0.000018] Switching to timer-based delay loop, resolution 41ns
[    0.000205] Console: colour dummy device 80x30
[    0.000256] Calibrating delay loop (skipped), value calculated using timer frequency.. 48.00 BogoMIPS (lpj=240000)
[    0.000268] pid_max: default: 32768 minimum: 301
[    0.000435] Mount-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
[    0.000447] Mountpoint-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
[    0.001187] CPU: Testing write buffer coherency: ok
[    0.001522] /cpus/cpu@0 missing clock-frequency property
[    0.001541] /cpus/cpu@1 missing clock-frequency property
[    0.001557] /cpus/cpu@2 missing clock-frequency property
[    0.001574] /cpus/cpu@3 missing clock-frequency property
[    0.001584] CPU0: thread -1, cpu 0, socket 0, mpidr 80000000
[    0.002095] Setting up static identity map for 0x40100000 - 0x40100060
[    0.002210] rcu: Hierarchical SRCU implementation.
[    0.002693] smp: Bringing up secondary CPUs ...
[    0.013436] CPU1: thread -1, cpu 1, socket 0, mpidr 80000001
[    0.024285] CPU2: thread -1, cpu 2, socket 0, mpidr 80000002
[    0.035042] CPU3: thread -1, cpu 3, socket 0, mpidr 80000003
[    0.035128] smp: Brought up 1 node, 4 CPUs
[    0.035147] SMP: Total of 4 processors activated (192.00 BogoMIPS).
[    0.035153] CPU: All CPU(s) started in HYP mode.
[    0.035157] CPU: Virtualization extensions available.
[    0.035709] devtmpfs: initialized
[    0.040566] VFP support v0.3: implementor 41 architecture 2 part 30 variant 7 rev 5
[    0.040815] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 19112604462750000 ns
[    0.040841] futex hash table entries: 1024 (order: 4, 65536 bytes, linear)
[    0.041668] pinctrl core: initialized pinctrl subsystem
[    0.042850] NET: Registered protocol family 16
[    0.044183] DMA: preallocated 256 KiB pool for atomic coherent allocations
[    0.045128] thermal_sys: Registered thermal governor 'step_wise'
[    0.045585] hw-breakpoint: found 5 (+1 reserved) breakpoint and 4 watchpoint registers.
[    0.045602] hw-breakpoint: maximum watchpoint size is 8 bytes.
[    0.064527] SCSI subsystem initialized
[    0.065025] usbcore: registered new interface driver usbfs
[    0.065084] usbcore: registered new interface driver hub
[    0.065147] usbcore: registered new device driver usb
[    0.065330] mc: Linux media interface: v0.10
[    0.065360] videodev: Linux video capture interface: v2.00
[    0.065447] pps_core: LinuxPPS API ver. 1 registered
[    0.065453] pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
[    0.065472] PTP clock support registered
[    0.065901] Advanced Linux Sound Architecture Driver Initialized.
[    0.066730] clocksource: Switched to clocksource arch_sys_counter
[    0.073729] NET: Registered protocol family 2
[    0.074256] tcp_listen_portaddr_hash hash table entries: 512 (order: 0, 6144 bytes, linear)
[    0.074286] TCP established hash table entries: 4096 (order: 2, 16384 bytes, linear)
[    0.074329] TCP bind hash table entries: 4096 (order: 3, 32768 bytes, linear)
[    0.074391] TCP: Hash tables configured (established 4096 bind 4096)
[    0.074517] UDP hash table entries: 256 (order: 1, 8192 bytes, linear)
[    0.074563] UDP-Lite hash table entries: 256 (order: 1, 8192 bytes, linear)
[    0.074756] NET: Registered protocol family 1
[    0.075362] RPC: Registered named UNIX socket transport module.
[    0.075375] RPC: Registered udp transport module.
[    0.075380] RPC: Registered tcp transport module.
[    0.075385] RPC: Registered tcp NFSv4.1 backchannel transport module.
[    0.076871] hw perfevents: enabled with armv7_cortex_a7 PMU driver, 5 counters available
[    0.077917] Initialise system trusted keyrings
[    0.078130] workingset: timestamp_bits=30 max_order=17 bucket_order=0
[    0.084084] NFS: Registering the id_resolver key type
[    0.084127] Key type id_resolver registered
[    0.084134] Key type id_legacy registered
[    0.084185] Key type asymmetric registered
[    0.084194] Asymmetric key parser 'x509' registered
[    0.084243] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 246)
[    0.084252] io scheduler mq-deadline registered
[    0.084259] io scheduler kyber registered
[    0.084959] sun4i-usb-phy 1c19400.phy: Couldn't request ID GPIO
[    0.088945] sun8i-h3-pinctrl 1c20800.pinctrl: initialized sunXi PIO driver
[    0.090645] sun8i-h3-r-pinctrl 1f02c00.pinctrl: initialized sunXi PIO driver
[    0.137305] Serial: 8250/16550 driver, 8 ports, IRQ sharing disabled
[    0.139163] sun8i-h3-pinctrl 1c20800.pinctrl: supply vcc-pa not found, using dummy regulator
[    0.140139] printk: console [ttyS0] disabled
[    0.160346] 1c28000.serial: ttyS0 at MMIO 0x1c28000 (irq = 34, base_baud = 1500000) is a U6_16550A
[    0.829587] printk: console [ttyS0] enabled
[    0.834372] sun8i-h3-pinctrl 1c20800.pinctrl: supply vcc-pg not found, using dummy regulator
[    0.863773] 1c28400.serial: ttyS1 at MMIO 0x1c28400 (irq = 35, base_baud = 1500000) is a U6_16550A
[    0.878577] lima 1c40000.gpu: gp - mali400 version major 1 minor 1
[    0.884801] lima 1c40000.gpu: pp0 - mali400 version major 1 minor 1
[    0.891144] lima 1c40000.gpu: pp1 - mali400 version major 1 minor 1
[    0.897455] lima 1c40000.gpu: l2 cache 64K, 4-way, 64byte cache line, 64bit external bus
[    0.905911] lima 1c40000.gpu: bus rate = 200000000
[    0.910740] lima 1c40000.gpu: mod rate = 297000000
[    0.915608] lima 1c40000.gpu: dev_pm_opp_set_regulators: no regulator (mali) found: -19
[    0.924065] lima 1c40000.gpu: Failed to register cooling device
[    0.930385] [drm] Initialized lima 1.1.0 20191231 for 1c40000.gpu on minor 0
[    0.938962] libphy: Fixed MDIO Bus: probed
[    0.943379] CAN device driver interface
[    0.947873] ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
[    0.954392] ehci-platform: EHCI generic platform driver
[    0.959848] ehci-platform 1c1a000.usb: EHCI Host Controller
[    0.965440] ehci-platform 1c1a000.usb: new USB bus registered, assigned bus number 1
[    0.973471] ehci-platform 1c1a000.usb: irq 28, io mem 0x01c1a000
[    1.006745] ehci-platform 1c1a000.usb: USB 2.0 started, EHCI 1.00
[    1.013608] hub 1-0:1.0: USB hub found
[    1.017417] hub 1-0:1.0: 1 port detected
[    1.021794] ohci_hcd: USB 1.1 'Open' Host Controller (OHCI) Driver
[    1.028006] ohci-platform: OHCI generic platform driver
[    1.033419] ohci-platform 1c1a400.usb: Generic Platform OHCI controller
[    1.040059] ohci-platform 1c1a400.usb: new USB bus registered, assigned bus number 2
[    1.047992] ohci-platform 1c1a400.usb: irq 29, io mem 0x01c1a400
[    1.121434] hub 2-0:1.0: USB hub found
[    1.125215] hub 2-0:1.0: 1 port detected
[    1.133358] sun6i-rtc 1f00000.rtc: registered as rtc0
[    1.138469] sun6i-rtc 1f00000.rtc: setting system clock to 1970-01-01T01:45:21 UTC (6321)
[    1.146636] sun6i-rtc 1f00000.rtc: RTC enabled
[    1.151328] i2c /dev entries driver
[    1.155951] sun8i-di 1400000.deinterlace: Device registered as /dev/video0
[    1.164269] sunxi-wdt 1c20ca0.watchdog: Watchdog enabled (timeout=16 sec, nowayout=0)
[    1.172509] sun8i-h3-r-pinctrl 1f02c00.pinctrl: supply vcc-pl not found, using dummy regulator
[    1.181842] sun8i-h3-pinctrl 1c20800.pinctrl: supply vcc-pf not found, using dummy regulator
[    1.191168] sunxi-mmc 1c0f000.mmc: Got CD GPIO
[    1.219375] sunxi-mmc 1c0f000.mmc: initialized, max. request size: 16384 KB
[    1.227405] sunxi-mmc 1c10000.mmc: allocated mmc-pwrseq
[    1.255908] sunxi-mmc 1c10000.mmc: initialized, max. request size: 16384 KB
[    1.263929] sun8i-ce 1c15000.crypto: Set mod clock to 50000000 (50 Mhz) from 24000000 (24 Mhz)
[    1.272764] sun8i-ce 1c15000.crypto: will run requests pump with realtime priority
[    1.280628] sun8i-ce 1c15000.crypto: will run requests pump with realtime priority
[    1.288429] sun8i-ce 1c15000.crypto: will run requests pump with realtime priority
[    1.296163] sun8i-ce 1c15000.crypto: will run requests pump with realtime priority
[    1.303904] sun8i-ce 1c15000.crypto: Register cbc(aes)
[    1.309194] sun8i-ce 1c15000.crypto: Register ecb(aes)
[    1.309279] mmc0: host does not support reading read-only switch, assuming write-enable
[    1.314443] sun8i-ce 1c15000.crypto: Register cbc(des3_ede)
[    1.324143] mmc0: new high speed SDXC card at address 59b4
[    1.328004] sun8i-ce 1c15000.crypto: Register ecb(des3_ede)
[    1.334197] mmcblk0: mmc0:59b4 USD00 118 GiB
[    1.339197] sun8i-ce 1c15000.crypto: CryptoEngine Die ID 1
[    1.349451] mmc1: queuing unknown CIS tuple 0x80 (2 bytes)
[    1.350018] usbcore: registered new interface driver usbhid
[    1.355902]  mmcblk0: p1 p2
[    1.360544] usbhid: USB HID core driver
[    1.364819] mmc1: queuing unknown CIS tuple 0x80 (3 bytes)
[    1.369624] cedrus 1c0e000.video-codec: Device registered as /dev/video1
[    1.374164] mmc1: queuing unknown CIS tuple 0x80 (3 bytes)
[    1.383659] NET: Registered protocol family 17
[    1.387552] mmc1: queuing unknown CIS tuple 0x80 (7 bytes)
[    1.389328] can: controller area network core (rev 20170425 abi 9)
[    1.398039] mmc1: queuing unknown CIS tuple 0x81 (9 bytes)
[    1.401043] NET: Registered protocol family 29
[    1.410852] can: raw protocol (rev 20170425)
[    1.415118] can: broadcast manager protocol (rev 20170425 t)
[    1.420788] can: netlink gateway (rev 20190810) max_hops=1
[    1.426503] Key type dns_resolver registered
[    1.430869] Registering SWP/SWPB emulation handler
[    1.435697] Loading compiled-in X.509 certificates
[    1.451988] random: fast init done
[    1.452812] usb_phy_generic usb_phy_generic.1.auto: supply vcc not found, using dummy regulator
[    1.465075] musb-hdrc musb-hdrc.2.auto: MUSB HDRC host driver
[    1.470875] musb-hdrc musb-hdrc.2.auto: new USB bus registered, assigned bus number 3
[    1.479594] hub 3-0:1.0: USB hub found
[    1.483378] hub 3-0:1.0: 1 port detected
[    1.491148] cfg80211: Loading compiled-in X.509 certificates for regulatory database
[    1.492749] mmc1: new high speed SDIO card at address 0001
[    1.501894] cfg80211: Loaded X.509 cert 'sforshee: 00b28ddf47aef9cea7'
[    1.511117] ALSA device list:
[    1.514087]   No soundcards found.
[    1.518149] platform regulatory.0: Direct firmware load for regulatory.db failed with error -2
[    1.526788] cfg80211: failed to load regulatory.db
[    1.539726] EXT4-fs (mmcblk0p2): mounted filesystem with ordered data mode. Opts: (null)
[    1.548847] VFS: Mounted root (ext4 filesystem) readonly on device 179:2.
[    1.556145] devtmpfs: mounted
[    1.581429] Freeing unused kernel memory: 1024K
[    1.650103] Run /sbin/init as init process
[    1.807097] EXT4-fs (mmcblk0p2): re-mounted. Opts: (null)
Starting syslogd: OK
Starting klogd: OK
Running sysctl: OK
Seeding 2048 bits and crediting
[    2.019800] random: crng init done
Saving 2048 bits of creditable seed for next boot
Starting network: [    2.156667] brcmfmac: brcmf_fw_alloc_request: using brcm/brcmfmac43430-sdio for chip BCM43430/1
[    2.330390] brcmfmac: brcmf_fw_alloc_request: using brcm/brcmfmac43430-sdio for chip BCM43430/1
[    2.347368] brcmfmac: brcmf_c_preinit_dcmds: Firmware: BCM43430/1 wl0: Mar 30 2021 01:12:21 version 7.45.98.118 (7d96287 CY) FWID 01-32059766
 .
Successfully initialized wpa_supplicant
rfkill: Cannot open RFKILL control device
udhcpc: started, v1.36.1
udhcpc: broadcasting discover
udhcpc: no lease, forking to background
OK
Starting dhcpcd...
dhcpcd-10.0.1 starting
sandbox unavailable: seccomp
DUID 00:01:00:01:c7:92:bc:d6:ac:6a:a3:29:f1:05
sandbox unavailable: seccomp
wlan0: connected to Access Point: ?????
wlan0: IAID a3:29:f1:05
wlan0: rebinding lease of 192.168.178.76
wlan0: probing address 192.168.178.76/24
wlan0: leased 192.168.178.76 for 864000 seconds
wlan0: adding route to 192.168.178.0/24
wlan0: adding default route via 192.168.178.1
forked to background, child pid 157
Starting nginx...

Welcome to Buildroot for the Bananapi M2 Zero
buildroot login: root
```
## Load the Driver
Load the driver using the 'modprobe' command. The loading process can be automated by including it in an init script: 
```console
# modprobe brcmfmac
[   67.333595] brcmfmac: brcmf_fw_alloc_request: using brcm/brcmfmac43430-sdio for chip BCM43430/1
[   67.513865] brcmfmac: brcmf_fw_alloc_request: using brcm/brcmfmac43430-sdio for chip BCM43430/1
[   67.530610] brcmfmac: brcmf_c_preinit_dcmds: Firmware: BCM43430/1 wl0: Mar 30 2021 01:12:21 version 7.45.98.118 (7d96287 CY) FWID 01-32059766
```

## Connect to WiFi Acces Point
In the final step, you simply need to create a file with your Wi-Fi access details and set an IP address: script:
```console
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

## Auto Connect to a Wifi Access Point
Once you created the file '/etc/wpa_supplicant.conf' and tested our wifi-connection, you can automate the driver-loading, connecting, and ip-assigning by a simply entry in the file '/etc/network/interfaces'
Yust add the following content into the file:
```text
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
