# NetBooting Compute Modules from Container Images

The advent of low-cost small-form-factor networked single-board computers (SBCs) such as the Raspberry Pi, and in particular the even smaller "compute module" variants, which must be coupled with an I/O board, means a low-powered cluster is now within the reach of even the most modest of hobbyist budgets. Boards such as the DeskPi Super6C, which host six Raspberry Pi CM4-compatible compute modules on a single mini-ITX baseboard and provides an internal gigabit switch between them, along with SD card and M.2 NVMe slots, bring new meaning to "high-denisty computing".

However, once you have more than a handful of compute nodes, flashing OS images onto them or burning SD cards can become tedious. Whilst this isn't necessarily something you need to do often in production, it can be a real burden when prototyping. Network booting (NetBooting) is the solution to that challenge: when powered on, each node contacts a server on the local network to download configuration information, a kernel image, and optional ramdisk, which it then boots as though they were on an SD card.

NetBooting can be coupled with an NFS server from which the nodes can mount a root filesystem, which can then either be transferred to local storage, or used on an ongoing basis in a diskless or semi-diskless setup.

## NetBoot process

Each device works broadly the same way:-

1. The device brings up its first ethernet port and attempts to obtain an IPv4 address from a DHCP server
2. As well as the normal IPv4 configuration details, the response from the DHCP server includes information that directs the device to a TFTP server
3. The TFTP server contains the kernel, ramdisk, and config files the device needs

### Raspberry Pi

Raspberry Pi devices with on-board Ethernet and Compute Modules all have NetBoot capability which they can fall back to if local boot methods (MMC, SD, NVMe, USB) fail and CM4s will behave like this by default. Newer Raspberry Pi 400 models (and potentially others) will default to a "Net Install" option rather than NetBooting, and will need to be booted into Linux and the boot order settings in the EEPROM updated using the `rpi-eeprom-config` utility [as documented on the official site](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#raspberry-pi-bootloader-configuration).

To boot a Raspberry Pi, the following options are required (for ISC DHCP server):

```
  option vendor-encapsulated-options "Raspberry Pi Boot";
  option vendor-class-identifier "PXEClient";
  option tftp-server-name "192.168.150.1";
```

Where `192.168.150.1` should be replaced with the IPv4 address of the TFTP server.

RPi devices will prefer to obtain files from a subdirectory on the TFTP server whose name matches their serial number. The serial number is displayed on the POST screen (the second hex number on the line starting `Board`), and in `/proc/cpuinfo` under Linux (the bottom half of the value under `Serial`). If `<serial>/config.txt` is found, then all subsequent requests for files will be prefixed with `<serial>/`, otherwise it will fall back to the TFTP server's root directory.

The files in the TFTP server directory are identical to those normally placed on the VFAT partition of a bootable SD card for the Pi. Upstream versions can be obtained from the [Raspberry Pi firmware repository](https://github.com/raspberrypi/firmware) (**Caution**: that repository is 25GB if cloned).


