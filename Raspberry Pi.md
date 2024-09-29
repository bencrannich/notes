# Raspberry Pi

## Hardware notes

### Compute Modules

The Raspberry Pi Compute Module 4 (CM4) is a small form-factor device equivalent to the Raspberry Pi 4b, but without any user-facing I/O connectors—instead, a pair of 100-pin edge connectors are used to mate it with a baseboard. Baseboards and alternative electrically-compatible compute modules are available from various manufacturers.

[Extensive documentation](https://www.raspberrypi.com/documentation/computers/compute-module.html) is provided about the CM4, including its boot process (which is compatible with other Raspberry Pi models).

### Pi 400

The Raspberry Pi 400 is a Raspberry Pi 4b integrated into a keyboard case with an integrated SD card slot, GPIO header, and a pair of mini-HDMI outputs.

## EEPROM updates

When updating the EEPROM, for example to modify the [bootloader configuration](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#raspberry-pi-bootloader-configuration), the device must be allowed to reboot back into the Linux installation performing the update, because it's not actually performed until after that reboot.
