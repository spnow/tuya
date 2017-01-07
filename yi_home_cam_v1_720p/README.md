# Road To Hacking Yi Cam Home 720P - Night Vision (17CN)

## Check List

1. USB TO RS232 UART SERIAL CONVERTER 
2. Soldering Station Set
3. Rainbow Cable
4. Multi-meter
5. If you screw everything up (which i did), you will need a Bus Pirate

## Intro

I got this cam from China, first things first. The cam only connect to the China (China version) app. Android is a APK and IOS you need to get from the China Apple App Store.

After sorting out all the mess. The Cam actually TALK!, It says "This cam can only works within China"

The worse of all, this cam is not downgradeable and not seems to be able to upgrade too. Maybe just me.

So, is this the End? NO, Hacking starts here.

## Part I: Gettings the Console

Open up the cover, hunt for (soldering skill needed) 
- RX
- TX
- GND

connect to your USB to RS232 converter and you will get a console with root. Not Hard.

## Network settings

> /etc/init.d # cat /home/conf/wpa_supplicant.conf
```
ctrl_interface=/var/run/wpa_supplicant
ap_scan=1
network={
ssid="MY_WIFI_L4H"
scan_ssid=1
proto=WPA RSN
key_mgmt=WPA-PSK
pairwise=CCMP TKIP
group=CCMP TKIP
psk="my_PASSWORD_l4h"
}
```

After we manage to setup the Wifi, we need few things for us to get easy access,

1. Telnet
2. FTP
3. RTSP

All these 3 deamons needs a busybox, so we google says there is a complete version from XiaoYi busybox (download from this folder) which allow us to to run these 3 deamons. Download, put into SD, mount it and put into the /home

1. Sym link a telnetd in /home/app
2. Sym link a tcpsvd in /home/app

## Bring up some services

> /etc/init.d # cat S88telnet
```
#!/bin/sh
/home/app/telnetd &
(sleep 10; /home/base/tools/wpa_supplicant -iwlan0 -c/home/conf/wpa_supplicant.conf) &
(sleep 20; /sbin/ifconfig wlan0 192.168.0.100 netmask 255.255.255.0) &
```

> /etc/init.d # cat S89ftp
```
#!/bin/sh
/home/app/tcpsvd -vE 0.0.0.0 21 ftpd -w / &
```

## RTSP returns segmentation fault

Fire up IDA pro and look at the RTSP Binary, we found few files requred before it can run, so this is how we fix it.

```
ln -s /tmp/hd1 /home/hd1
ln -s /tmp/hd2 /home/hd2
ln -s /tmp /home/mmap_tmpfs
mkdir /home/jrview
ln -s /home/app/busybox /bin/renice
```

Till this stage, 

- WiFi - Up, Tested
- telnetd - Up, Tested
- ftpd - Up, Tested
- rtsp - Up, yet to test

With not enough coffee consumption I decided to delete the /bin/busybox and and want to copy the full buxybox into /bin/busybox. Wihout checking

- Space in rootfs is way too small
- without backing up the original busybox

Again I rebooted the box without a proper check. Kernel Panic is the gift.

## The Kernel Panic
```
## Booting kernel from Legacy Image at 81000000 ...
   Image Name:   7518-hi3518c-kernel
   Image Type:   ARM Linux Kernel Image (uncompressed)
   Data Size:    1385848 Bytes = 1.3 MiB
   Load Address: 80008000
   Entry Point:  80008000
   Loading Kernel Image ... OK
OK

Starting kernel ...

Uncompressing Linux... done, booting the kernel.
Booting Linux on physical CPU 0
Linux version 3.4.35 (chenshibo@ANTS-SH-SV02) (gcc version 4.8.3 20131202 (prerelease) (Hisilicon_v300) ) #3 Tue May 31 17:35:37 CST 2016
CPU: ARM926EJ-S [41069265] revision 5 (ARMv5TEJ), cr=00053177
CPU: VIVT data cache, VIVT instruction cache
Machine: hi3518ev200
Memory policy: ECC disabled, Data cache writeback
Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 5588
Kernel command line: mem=22M console=ttyAMA0,115200 root=/dev/mtdblock4 rootfstype=jffs2 mtdparts=hi_sfc:256k(boot)ro,64k(env),64k(conf),1600k(os),1280k(rootfs),12992k(home),64k(vd1),64k(ver)
PID hash table entries: 128 (order: -3, 512 bytes)
Dentry cache hash table entries: 4096 (order: 2, 16384 bytes)
Inode-cache hash table entries: 2048 (order: 1, 8192 bytes)
Memory: 22MB = 22MB total
Memory: 18480k/18480k available, 4048k reserved, 0K highmem
Virtual kernel memory layout:
    vector  : 0xffff0000 - 0xffff1000   (   4 kB)
    fixmap  : 0xfff00000 - 0xfffe0000   ( 896 kB)
    vmalloc : 0xc1800000 - 0xff000000   ( 984 MB)
    lowmem  : 0xc0000000 - 0xc1600000   (  22 MB)
    modules : 0xbf000000 - 0xc0000000   (  16 MB)
      .text : 0xc0008000 - 0xc0340000   (3296 kB)
      .init : 0xc0340000 - 0xc035cd04   ( 116 kB)
      .data : 0xc035e000 - 0xc037f360   ( 133 kB)
       .bss : 0xc037f384 - 0xc03b1850   ( 202 kB)
SLUB: Genslabs=13, HWalign=32, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
NR_IRQS:32
VIC @fe0d0000: id 0x00641190, vendor 0x41
sched_clock: 32 bits at 49MHz, resolution 20ns, wraps every 86767ms
Console: colour dummy device 80x30
Calibrating delay loop... 298.59 BogoMIPS (lpj=1492992)
pid_max: default: 32768 minimum: 301
Mount-cache hash table entries: 512
Initializing cgroup subsys freezer
CPU: Testing write buffer coherency: ok
Setting up static identity map for 0x8026ab20 - 0x8026ab78
dummy:
NET: Registered protocol family 16
Serial: AMBA PL011 UART driver
uart:0: ttyAMA0 at MMIO 0x20080000 (irq = 5) is a PL011 rev2
console [ttyAMA0] enabled
uart:1: ttyAMA1 at MMIO 0x20090000 (irq = 30) is a PL011 rev2
uart:2: ttyAMA2 at MMIO 0x200a0000 (irq = 25) is a PL011 rev2
bio: create slab <bio-0> at 0
SCSI subsystem initialized
usbcore: registered new interface driver usbfs
usbcore: registered new interface driver hub
usbcore: registered new device driver usb
Switching to clocksource timer0
NET: Registered protocol family 2
IP route cache hash table entries: 1024 (order: 0, 4096 bytes)
TCP established hash table entries: 1024 (order: 1, 8192 bytes)
TCP bind hash table entries: 1024 (order: 0, 4096 bytes)
TCP: Hash tables configured (established 1024 bind 1024)
TCP: reno registered
UDP hash table entries: 256 (order: 0, 4096 bytes)
UDP-Lite hash table entries: 256 (order: 0, 4096 bytes)
NET: Registered protocol family 1
jffs2: version 2.2. (NAND) © 2001-2006 Red Hat, Inc.
fuse init (API version 7.18)
msgmni has been set to 36
io scheduler noop registered
io scheduler deadline registered (default)
io scheduler cfq registered
brd: module loaded
Check Flash Memory Controller v100 ...  Found.
SPI Nor(cs 0) ID: 0xc8 0x40 0x18
Block:64KB Chip:16MB Name:"GD25Q128"
SPI Nor total size: 16MB
8 cmdlinepart partitions found on MTD device hi_sfc
8 cmdlinepart partitions found on MTD device hi_sfc
Creating 8 MTD partitions on "hi_sfc":
0x000000000000-0x000000040000 : "boot"
0x000000040000-0x000000050000 : "env"
0x000000050000-0x000000060000 : "conf"
0x000000060000-0x0000001f0000 : "os"
0x0000001f0000-0x000000330000 : "rootfs"
0x000000330000-0x000000fe0000 : "home"
0x000000fe0000-0x000000ff0000 : "vd1"
0x000000ff0000-0x000001000000 : "ver"
ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
hiusb-ehci hiusb-ehci.0: HIUSB EHCI
hiusb-ehci hiusb-ehci.0: new USB bus registered, assigned bus number 1
hiusb-ehci hiusb-ehci.0: irq 15, io mem 0x100b0000
hiusb-ehci hiusb-ehci.0: USB 0.0 started, EHCI 1.00
hub 1-0:1.0: USB hub found
hub 1-0:1.0: 1 port detected
i2c /dev entries driver
hisi_i2c hisi_i2c.0: Hisilicon [i2c-0] probed!
hisi_i2c hisi_i2c.1: Hisilicon [i2c-1] probed!
hisi_i2c hisi_i2c.2: Hisilicon [i2c-2] probed!
TCP: cubic registered
Initializing XFRM netlink socket
NET: Registered protocol family 17
NET: Registered protocol family 15
lib80211: common routines for IEEE802.11 drivers
Registering the dns_resolver key type
VFS: Mounted root (jffs2 filesystem) on device 31:4.
Freeing init memory: 112K
Kernel panic - not syncing: No init found.  Try passing init= option to kernel. See Linux Documentation/init.txt for guidance.
```





## Part II, Taking over the flash room




## Taking Partition Notes

Partition by size
```
0x000000000000-0x000000040000 : "boot"
0x000000040000-0x000000050000 : "env"
0x000000050000-0x000000060000 : "conf"
0x000000060000-0x0000001f0000 : "os"
0x0000001f0000-0x000000330000 : "rootfs"
0x000000330000-0x000000fe0000 : "home"
0x000000fe0000-0x000000ff0000 : "vd1"
0x000000ff0000-0x000001000000 : "ver"
```


## Dump using bus pirate
```
flashrom -p buspirate_spi:dev=/dev/ttyUSB0 -c GD25Q128C -r yicam_night_GD25Q128C.bin -V -f
```


This is how you split the file according to partition size
```
dd if=yicam_night_test_GD25Q128C.bin of=yicam_night_test_GD25Q128C_bootloader.bin bs=1 count=$((0x040000))
dd if=yicam_night_test_GD25Q128C.bin of=yicam_night_test_GD25Q128C_env.bin bs=1 count=$((0x050000-0x040000)) skip=$((0x040000))
dd if=yicam_night_test_GD25Q128C.bin of=yicam_night_test_GD25Q128C_conf.bin bs=1 count=$((0x060000-0x050000)) skip=$((0x050000))
dd if=yicam_night_test_GD25Q128C.bin of=yicam_night_test_GD25Q128C_os.bin bs=1 count=$((0x1f0000-0x060000)) skip=$((0x060000))
dd if=yicam_night_test_GD25Q128C.bin of=yicam_night_test_GD25Q128C_rootfs.bin bs=1 count=$((0x330000-0x1f0000)) skip=$((0x1f0000))
dd if=yicam_night_test_GD25Q128C.bin of=yicam_night_test_GD25Q128C_home.bin bs=1 count=$((0xfe0000-0x330000)) skip=$((0x330000))
dd if=yicam_night_test_GD25Q128C.bin of=yicam_night_test_GD25Q128C_vd1.bin bs=1 count=$((0xff0000-0xfe0000)) skip=$((0xfe0000))
dd if=yicam_night_test_GD25Q128C.bin of=yicam_night_test_GD25Q128C_ver.bin bs=1 count=$((0x1000000-0xff0000)) skip=$((0xff0000))
```



```

Just In case you need padding before write
```
ruby -e 'print "\xFF" * 393216' >> rootfs_e.jjfs
```
Merging into JJFS
```
(dd if=yicam_night_test_GD25Q128C_bootloader.bin ) > yicam_full_e.bin
(dd if=yicam_night_test_GD25Q128C_env.bin ) >> yicam_full_e.bin
(dd if=yicam_night_test_GD25Q128C_conf.bin ) >> yicam_full_e.bin
(dd if=yicam_night_test_GD25Q128C_os.bin ) >> yicam_full_e.bin
(dd if=yicam_night_test_GD25Q128C_rootfs_e.bin ) >> yicam_full_e.bin
(dd if=yicam_night_test_GD25Q128C_home.bin ) >> yicam_full_e.bin
(dd if=yicam_night_test_GD25Q128C_vd1.bin ) >> yicam_full_e.bin
(dd if=yicam_night_test_GD25Q128C_ver.bin ) >> yicam_full_e.bin
```


