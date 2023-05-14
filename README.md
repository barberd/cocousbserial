# CoCo USB Serial Cartridge

## Description

This board is a cartridge for the Tandy Radio Shack TRS-80 Color Computer (CoCo), an 8 bit computer produced between 1980 and 1991. It provides an interface that allows for a serial connection over a USB connection to a host computer (typically a modern PC).

![Front View](images/cocousbserial-front.png?raw=true)

Schematic is available [here](kicad/6551usb.pdf).

## Source and License

Design maintained at [https://github.com/barberd/cocousbserial](https://github.com/barberd/cocousbserial). [Kicad](https://www.kicad.org/) and [Freerouting](https://github.com/freerouting/freerouting/) were used to design the board.

The design is copyright 2023 by Don Barber. The design is open source, distributed via the GNU GPL version 3 license. Please see the COPYING file for details.

## Interface

The board replicates the interface for original CoCo devices based on the 6551 ACIA chip, such as the Deluxe RS-232 Pak and the Direct Connect Modem Pak. Most of the behavior of the 6551 is duplicated, including the status, command, and control registers. One exception is a lack of hardware loopback support; another is the reset values of the registers (see next paragraph). Interrupts are supported, including those triggered by an empty transmit buffer, received data, or a change in connection status. For the latter, changes in USB connection status are used instead of the DCD and DSR lines used in a rs-232 serial connection.

The registers are not set in the same way as the 6551 upon reset or power-on. As most software does its own initialization of the registers when starting anyway, the lack of this functionality has little practical effect. However, it is possible that the card receiving USB data before the software has set up (or disabled) interrupt handling might lead to an unhandled interrupt, which could potentially lock up the system. The easiest workaround is to just not send data from the USB host until after the CoCo software has initialized the card.

Settings in the command or control registers that do not apply to a USB connection or are performed by the USB protocol (such as baud rate, parity, bitrate, and stop bit settings) are ignored. Bitflags in the status register that do not apply or are abstracted away by the USB module (such as parity, framing, or overrun errors) are tied to 0.

## Why duplicate a legacy hardware interface?

Duplicating the behavior of the 6551 in this way allows one to use original CoCo software written for the Deluxe RS-232 Pak or Direct Connect Modem Pak without modification. Additionally, the use of standard TTL chips (when combined with the USB module) allows one to build the board at home without the need for specialized chips such as a programmed microcontroller.

## USB Capability

The USB module used is the DLP-USB245R. [Here is its manufacturer's page](https://www.dlpdesign.com/usb/usb245r.php). The module can be purchased directly from the manufacturer or from electronics supply houses such as Jameco and Mouser. This module abstracts away much of the heavy lifting needed for USB communications. This module acts as a client device, communicating with a USB host, typically a modern PC. It does NOT support host mode, meaning it can not be used as a host for other peripherals such as keyboard and mice; it only provides a client serial connection. The DLP-USB1232H module is also pin-compatible with the board, though one will need to configure that module into DLP-USB245R (FIFO) mode using the manufacturer's configuration software.

## IO Address

The 6 dip switches (SW1) set the IO addresses for the board. These correspond to address lines A2 through A7. The four addresses used correspond to the board's (and the original 6551 chip's) four registers. Using the switches, one can configure the base address of the board to be any multiple of four within the range of FF00 through FFFC.

The Deluxe RS-232 Pak uses IO addresses FF68-FF6B while the Direct Connect Modem Pak uses FF6C-FF6F. Markings on the board make it straightforward to set the IO address of the board to clone either; this is the easiest way to use the board as one can then leverage existing software.

One may choose to use a different IO address range; this can be helpful if one has multiple paks that would otherwise conflict. If you use a non-standard range, software must be configured, patched, or updated to use the new IO address. Be mindful to review the CoCo's IO mapping and not choose an IO address used by another device, such as a keyboard, joystick, cassette, or disk controller.

## Power Draw

When hooked up to a bench power supply, the card drew 0.2 Amps.

Board version 1.09 added a jumper for the source of +5v power for the DLP-USB245R module, either the CoCo or the USB host. The module draws 15mA of power in normal operation. One should choose the USB host if one wants the module recognized by the host as soon as its plugged in, or wants the USB device to remain configured on the host even when the CoCo is reset. One should choose the CoCo if one wants the device recognized only when the CoCo is powered on.

## ROM Socket

A socket for a ROM is also included in the board design. This is entirely optional, as most users today will instead choose to use a third-party communications software package loaded from another source such as disk, SDC, or DriveWire. However, one can add a ROM loaded with the Deluxe RS-232 Cartridge or Direct Connect Modem Cartridge software (or any other cartridge software) if one desires to replicate the original cartridge experience.

The ROM socket is wired for a 27128 EPROM. As the 27128 provides 16k of storage, the board presents this as 2 different banks of 8k each, selected via jumper J1. As such, when writing the EPROM, combine the images with each image aligned to an 8k boundary (eg, at 0 and 8k). If using a ROM IC other than the 27128, one will need to adjust the board design to match the chosen IC. Some other ROM ICs are pin-compatible and will work fine with the existing design.

### Starting the ROM software

Please note that since the cartridge interrupt line is used for serial communications interrupts and not for the cartridge start interrupt, this cartridge does not auto-start. As such, one will need to start the ROM software manually with an "EXEC &HC000" or "EXEC 49152" statement, just as described in the [Deluxe RS-232 Pak](https://colorcomputerarchive.com/repo/Documents/Manuals/Hardware/Deluxe%20RS-232%20Operation%20Manual%20%28Tandy%29.pdf) and [Direct Connect Modem Pak](https://colorcomputerarchive.com/repo/Documents/Manuals/Hardware/Direct%20Connect%20Modem%20Pak%20%28Tandy%29.pdf) manuals.

Note the Color Computer 3 maps the cartridge memory differently than the CoCo 1 and CoCo 2; use "EXEC 57360" or "EXEC &HE010" instead.

## Physical case

The board is sized to fit into an original large CoCo pak, such as the Deluxe RS-232 Pak or a Disk Controller Pak. One can also 3D print a case such as [this  one found on thingiverse](https://www.thingiverse.com/thing:4829413).

## How to order for fabrication

Download 6551usb-fabrication.zip, then upload it to your PCB manufacturer of choice when asked to provide Gerber files. Usually this is found under a 'Quote' option on the website. Search "pcb manufacturing" on any major search engine to get several manufacturers.

Some may have ordered boards and have extra available. Reach out to don &#x40; dgb3.net to explore this.

## Errata

If you have version 1.04 or 1.05 of the board, there a 'bug' in the reset circuit for the change-in-plugged-in-status interrupt in that the interrupt is not cleared when the status register is read. I'm not aware of any software that actually enables this in the first place, so one will likely never encouter the bug. But the fix is simple: bend up pin 6 on U12, and solder it to pin 6 on U5 with a bodge wire. R4 is also no longer needed. This is fixed in 1.06.

Boards prior to versions 1.08 were printed with 10nF capacitors, but .1uf (100nF) capacitors are bit more standard. Either work fine, they're just to smooth out ripples in the power to the ICs.

Board version 1.09 added a jumper for the source of +5v power; see 'Power Draw' section above.
