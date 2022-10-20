# CoCo USB Serial Cartridge

## Description

This board is a cartridge for the Tandy Radio Shack TRS-80 Color Computer (CoCo), an 8 bit computer produced between 1980 and 1991. It provides an interface that allows for a serial connection over a USB connection to a host computer (typically a modern PC).

![Front View](images/cocousbserial-front.png?raw=true)

## Source and License

Design maintained at https://github.com/barberd/cocousbserial

The design is copyright 2022 by Don Barber. The design is open source, distributed via the GNU GPL version 3 license. Please see the COPYING file for details.

## Interface

The board replicates the interface for original CoCo devices based on the 6551 ACIA chip, such as the Deluxe RS-232 Pak and the Direct Connect Modem Pak. Most of the behavior of the 6551 is duplicated, including the status, command, and control registers. One exception is a lack of hardware loopback support. Interrupts are supported, including those triggered by an empty transmit buffer, received data, or a change in connection status. For the latter, changes in USB connection status are used instead of the DCD and DSR lines used in a rs-232 serial connection.

Settings in the command or control registers that do not apply to a USB connection or are performed by the USB protocol (such as baud rate, parity, bitrate, and stop bit settings) are ignored. Bitflags in the status register that do not apply or are abstracted away by the USB module (such as parity, framing, or overrun errors) are hard-coded to 0.

## Why duplicate a legacy hardware interface?

Duplicating the behavior of the 6551 in this way allows one to use original CoCo software written for the Deluxe RS-232 Pak or Direct Connect Modem Pak without modification. Additionally, the use of standard TTL chips (when combined with the USB module) allows one to build the board at home without the need for specialized chips such as a programmed microcontroller.

## USB Capability

The USB module used is the DLP-USB245R. Its datasheet is available at https://www.dlpdesign.com/usb/usb245r.php. This module abstracts away much of the heavy lifting needed for USB communications. This module acts as a client device, communicating with a USB host, typically a modern PC. It does NOT support host mode, meaning it can not be used as a host for other peripherals such as keyboard and mice; it only provides a client serial connection. The DLP-USB1232H module is also pin-compatible with the board, though one will need to configure that module into DLP-USB245R (FIFO) mode using the manufacturer's configuration software.

## ROM Socket

A socket for a ROM is also included in the board design. This is entirely optional, as most users today will instead choose to use a third-party communications software package loaded from another source such as disk, SDC, or DriveWire. However, one can add a ROM loaded with the Deluxe RS-232 Cartridge or Direct Connect Modem Cartridge software (or any other cartridge software) if one desires to replicate the original cartridge experience.

The ROM socket is wired for a 27128 EPROM. As the 27128 provides 16k of storage, the board presents this as 2 different banks of 8k each, selected via jumper J1. As such, when writing the EPROM, combine the images with each image aligned to an 8k boundary (eg, at 0, 8k). If using a ROM IC other than the 27128, one will need to adjust the board design to match the chosen IC.

### Starting the software

Please note that since the cartridge interrupt line is used for serial communications interrupts and not for the cartridge start interrupt, this cartridge does not auto-start. As such, one will need to start the ROM software manually with an "EXEC &HC000" or "EXEC 49152" statement, just as described in the Deluxe RS-232 Pak and Direct Connect Modem Pak manuals.

## Physical case

The board is sized to fit into an original large CoCo pak, such as the Deluxe RS-232 Pak or a Disk Controller Pak. One can also 3D print a case such as one found on thingiverse at https://www.thingiverse.com/thing:4829413.

