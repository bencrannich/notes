# NetBooting Compute Modules from Container Images

The advent of low-cost small-form-factor networked single-board computers (SBCs) such as the Raspberry Pi, and in particular the even smaller "compute module" variants, which must be coupled with an I/O board, means a low-powered cluster is now within the reach of even the most modest of hobbyist budgets. Boards such as the DeskPi Super6C, which host six Raspberry Pi CM4-compatible compute modules on a single mini-ITX baseboard and provides an internal gigabit switch between them, along with SD card and M.2 NVMe slots, bring new meaning to "low-cost high-density computing".

However, once you have more than a handful of compute nodes, flashing OS images onto them or burning SD cards can become tedious. Whilst this isn't necessarily something you need to do often in production, it can be a real burden when prototyping. Network booting (NetBooting) is the solution to that challenge: when powered on, each node contacts a server on the local network to download configuration information, a kernel image, and optional ramdisk, which it then boots as though they were on an SD card.

NetBooting can be coupled with an NFS server from which the nodes can mount a root filesystem, which can then either be transferred to local storage, or used on an ongoing basis in a diskless or semi-diskless setup.

## NetBoot process

Each device works broadly the same way:-

1. The device brings up its first ethernet port and attempts to obtain an IPv4 address from a DHCP server
2. As well as the normal IPv4 configuration details, the response from the DHCP server includes information that directs the device to a TFTP server
3. The TFTP server contains the kernel, ramdisk, and config files the device needs

### Raspberry Pi

Raspberry Pi devices with on-board Ethernet and Compute Modules all have NetBoot capability which they can fall back to if local boot methods (MMC, SD, NVMe, USB) fail and CM4s will behave like this by default. Newer Raspberry Pi 400 models (and potentially others) will default to a "Net Install" option rather than NetBooting, and will need to be booted into Linux and the boot order settings in the EEPROM updated using the `rpi-eeprom-config` utility [as documented on the official site](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#raspberry-pi-bootloader-configuration).

To boot a Raspberry Pi, the following options are required (for ISC DHCP server):—

```
  option vendor-encapsulated-options "Raspberry Pi Boot";
  option vendor-class-identifier "PXEClient";
  option tftp-server-name "192.168.150.1";
```

Where `192.168.150.1` should be replaced with the IPv4 address of the TFTP server.

RPi devices will prefer to obtain files from a subdirectory on the TFTP server whose name matches their serial number. The serial number is displayed on the POST screen (the second hex number on the line starting `Board`), and in `/proc/cpuinfo` under Linux (the bottom half of the value under `Serial`). If `<serial>/config.txt` is found, then all subsequent requests for files will be prefixed with `<serial>/`, otherwise it will fall back to the TFTP server's root directory.

The files in the TFTP server directory are identical to those normally placed on the VFAT partition of a bootable SD card for the Pi. Upstream versions can be obtained from the [Raspberry Pi firmware repository](https://github.com/raspberrypi/firmware) (**Caution**: that repository is 25GB if cloned).

### Milk-V Mars CM

The Milk-V Mars CM is a RISC-V 64 compute module with a form factor and basedboard connection compatible with the Raspberry Pi CM4, built around the same SoC as used in the StarFive VisionFive2, the JH7110. The Mars CM uses U-Boot and so can NetBoot, but won't by default.

The easiest way to modify the boot configuration on the Mars CM is to plug it into an I/O board with GPIO pins you have a UART attached to, it speaks 115200-8-n-1 almost instantly when powered on, and has a 3-second countdown before automatic boot commences.

The simplest possible way to configure a Mars CM to NetBoot by default is to interrupt automatic boot and then run:—

```
setenv bootcmd run dhcp_bootcmd
env save
```

If you execute the `reset` command, the CM will reboot itself, and then immediately fail because the DHCP server hasn't been configured properly. It won't by default retry.

U-Boot uses the DHCP `next-server` address as the TFTP server, accomplished via the following for ISC DHCP server:—

```
  next-server 192.168.150.1;
```

Optionally, the DHCP server can provide a `filename`, which is the name of a *kernel* image which will be loaded:

```
  filename "marscm/vmlinuz-6.6.20-starfive";
```

U-Boot will attempt to load a file named `XXX bootscr.uimg XXX` from the root of the TFTP server. This can be changed, for example to mimic the Raspberry Pi approach, via the following bootloader commands:-

```
setenv bootscript ABC123/bootscript.uimg
env save
```

Either the boot script or the kernel must be available. The bootscript must be prepared using the `mkimage` utility (included in the `u-boot-tools` Debian package):

```
mkimage -c none -A riscv -T script -d input.script bootscript.uimg
```

U-Boot is an extremely flexible and powerful bootloader and so the bootscript could do a great deal in principle (this needs to be balanced against maintainability).

A simple bootscript for the Mars CM would be:

```
tftpboot $kernel_addr_r /starfive/vmlinuz-6.6.20-starfive
tftpboot $fdt_addr_r /starfive/dtbs/6.6.20/starfive/jh7110-starfive-visionfive-2-v1.3b.dtb
tftpboot $ramdisk_addr_r /starfive/initrd.img-6.6.20-startfive
booti $kernel_addr_r $ramdisk_addr_r $fdt_addr_r
```

If you don't use an initrd, comment out the third `tftpboot` line and replace `$ramdisk_addr_r` with a hyphen in the `booti` line:

```
booti $kernel_addr_r - $fdt_addr_r
```
