# OpenBSD on PC Engines APU2

This is a writeup on getting OpenBSD running on PC Engines APU2.

Note: Some of the steps require downloading software. Whenever possible, verify checksums of such software or better yet, download it from a well-known repository. Remember [Reflections on Trusting Trust](https://www.ece.cmu.edu/~ganger/712.fall02/papers/p761-thompson.pdf)!

## Hardware

| Component  | Source | Link  | Price  |
|---|---|---|---|
| APU.2C4 system board (GX-412TC quad core / 4GB / 3 Intel GigE / 2 miniPCI express / mSATA / USB / RTC battery) | PC Engines | [apu2c4](http://www.pcengines.ch/apu2c4.htm) | $122
| Enclosure (3 LAN, black, USB) | PC Engines | [case1d2blku](http://www.pcengines.ch/case1d2blku.htm) | $10
| AC adapter 12V 2A euro plug | PC Engines | [ac12veur2 ](http://www.pcengines.ch/ac12veur2.htm) | $4.4
| SSD M-Sata 16GB MLC, Phison S9 controller | PC Engines | [msata16d](http://www.pcengines.ch/msata16d.htm) | $17
| SD card 4GB pSLC Phison (MLC flash running in SLC mode) | PC Engines, anywhere | [sd4b](http://www.pcengines.ch/sd4b.htm) | $6.6
| Compex WLE200NX 802.11a/b/g/n miniPCI express wireless card | PC Engines | [wle200nx](http://www.pcengines.ch/wle200nx.htm) | $19
| Pigtail cable I-PEX -> reverse SMA | PC Engines | [pigsma](http://www.pcengines.ch/pigsma.htm) | 2 x $1.5
| Antenna reverse SMA dual band | PC Engines | [antsmadb](http://www.pcengines.ch/antsmadb.htm) | 2 x $2.05
| Null modem cable DB9-F to DB9-F | PC Engines, anywhere | [db9cab1](http://www.pcengines.ch/db9cab1.htm) | $1.3
| USB to RS-232 DM9-M adapter | Sabrent (FTDI) | [USB-2920](https://www.sabrent.com/category/cables/USB-2920/) | $10.11 |
| 8GB USB Flash Drive | Amazon, anywhere | [CZ50](http://www.amazon.com/SanDisk-Cruzer-Frustration-Free-Packaging--SDCZ50-008G-AFFP/dp/B007KFAG7U/) | $4.29

Total $202.68 without shipping.

Assemble the hardware per the instructions on the PC Engines website. Remember:

  * Cooling
  * Ground wireless card
  * Insert the SD card

## Setup serial console

Note that the serial port settings for the APU2 are 115200 baud rate, 8N1 (8 data bits, no parity, 1 stop bit).

1. Install the USB to serial [driver](http://downloads.trendnet.com/tu-s9_v2/utilities/driver_tu-s9_20151110.zip)
2. Plug the USB end of the USB to serial cable to the Mac
3. Plug the DB9-M end of the USB to serial cable to one end of the null modem cable
4. Plug the other end of the null modem cable to the APU2
5. Connect to the APU2 from the terminal: `screen /dev/tty.usbserial 115200` (note: your device might be different, look for devices with `tty` and `serial` in their name)

Power the APU2 off and back on by pulling the plug and plugging it back in, respectively. You should see output on the screen.

## Bootable TinyCore Linux USB flash drive

BIOS updates require flashing the ROM. Create a bootable USB flash drive with TinyCore Linux from PC Engines. It includes `flashrom` but doesn't include any ROM images you might need. Steps:

1. Download TinyCore Linux ([apu2-tinycore6.4.img.gz](http://pcengines.ch/file/apu2-tinycore6.4.img.gz)) and extract it to get **apu2-tinycore6.4.img**
2. Download the latest ROM ([apu2_160311.zip](http://www.pcengines.ch/file/apu2_160311.zip)) and extract it to get **apu2_160311.rom**. It is required for making wireless networking work and booting from an SD card
3. Double click **apu2-tinycore6.4.img**
 to mount the image and drag **apu2_160311.rom** to it
4. Eject the TinyCore Linux image (usually named **SYSLINUX**)
5. Insert the USB flash drive to the Mac, figure out which device it is (with `diskutil list`, let's assume it's `/dev/disk2`) and unmount it (`diskutil unmountDisk /dev/disk2`)
6. Write the TinyCore Linux image to the flash drive: `sudo dd if=apu2-tinycore6.4.img of=/dev/rdisk2 bs=1m` (note the use of `rdisk2` - that's the raw device)
7. Eject the USB flash drive

## Update the BIOS

If the BIOS needs updating - the version displayed during boot is not the latest that's on the PC Engines website - follow these steps:

1. Power off the APU2
2. Insert the USB flash drive to one of the APU2's USB slots
3. Connect the serial console cable
4. Power on the APU2
5. Press F10 to enter the APU2 boot menu. In the boot menu, opt to boot from the USB flash drive (usually option number 1)
6. Once you get to a prompt, use `flashrom` to update the BIOS. The ROM file will be in `/media/SYSLINUX`: `flashrom -p internal -w /media/SYSLINUX/apu2_160311.rom`
7. When verification is done, reboot the APU2 so changes take effect

## Install OpenBSD

### Bootable OpenBSD installation USB flash drive

1. Download the OpenBSD installer, [`amd64/install59.fs`](http://ftp.openbsd.org/pub/OpenBSD/5.9/amd64/install59.fs) ([SHA256 fingerprint](http://ftp.openbsd.org/pub/OpenBSD/5.9/amd64/SHA256)), file-system image (not ISO!) from one of the mirrors
2. Insert the USB flash drive to the Mac, figure out which device it is (with `diskutil list`, let's assume it's `/dev/disk2`) and unmount it (`diskutil unmountDisk /dev/disk2`)
3. Write the installer image to the flash drive: `sudo dd if=install59.fs of=/dev/rdisk2 bs=1m` (note the use of `rdisk2` - that's the raw device)
4. Eject the USB flash drive

### Serial console settings for OpenBSD

The following settings are required for proper serial console output:

```
stty pc0 115200
stty com0 115200
set tty com0
```

Enter them in the `boot>` prompt when booting the installer. Put them in `/etc/boot.conf` so they're always in effect.

### Install OpenBSD

1. Power off the APU2
2. Insert the bootable OpenBSD installer USB flash drive to one of the USB slots on the APU2
3. Power on the APU2, press F10 to get to the boot menu, and choose to boot from USB (usually option number 1)
4. At the `boot>` prompt, remember the serial console settings (see above)
5. Also at the `boot>` prompt, press Enter to start the installer
6. Follow the installation instructions

### Firmware update

The driver used for wireless networking is [athn(4)](http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man4/athn.4?query=athn). It might not work properly out of the box. Once OpenBSD is installed, run `fw_update` with no arguments. It will figure out which firmware updates are required and will download and install them. When it finishes, `reboot`.

## Credits

These instructions were collected from websites, mailing lists, forums, and whatever I could find.
