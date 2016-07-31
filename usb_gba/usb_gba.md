---
title: USB to GBA
---

# USB to GBA #

## What you will find here ##
The usb_gba package provides software to transfer programs to the Gameboy Advance using its built-in multiboot protocol. All data transfer runs over USB, the Universal Serial Bus, which provides improved compatibility with several platforms and operating systems. In the future there might be serial interfacing software as well that will allow you to communicate through the serial port of the GBA.

## What you won't find here ##
Full-blown, commercial-like, ready to use software to plug-and-play with. The thing doesn't even come with a warranty. You use the given information at your own risk.

> This program is free software; you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation; either version 2 of the License, or (at your option) any later version. See also the file COPYING which came with this application.
This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License for more details.

## Where will you find it ##
- `multiboot/...`  
  Firmware for the AN2131 (aka EZ-USB) device that implements the multiboot protocol. Together with a host application, software for the GBA can be downloaded into 'external WRAM'.
- `apps/...`  
  Host applications which connect to the multiboot firmware on an EZ-USB device. Both command line and GUI versions are provided.
- `include/...`  
  Various include files needed to build the firmware and the host applications.

## What you will need ##
Hardware:

- apparantly a Gameboy Advance
- some kind of EZ-USB prototype board
  - <http://www.devasys.com/usbi2cio.htm>
  - <http://www.omnigroup.com/~wiml/soft/pic/keyspan.html>
- a cable to connect both, see [multiboot_schematic.pdf](multiboot_schematic.pdf)
- a computer wih USB

Software:

- the [usb_gba package](usb_gba-1.1.tar.gz) version 1.1 (here is [1.0](usb_gba-1.0.tar.gz))
- an operating system with USB support (preferably Linux, others might do as well)
- [libusb](http://libusb.sourceforge.net/) to handle all the USB interfacing stuff
- a C compiler (e.g. [gcc](http://www.gnu.org/)) to build the host application
- [gtk](http://www.gtk.org/), the Gimp Toolkit v1.2.x
- a firmware downloader to transfer the firmware to the EZ-USB proto board
  - <http://ezusb2131.sourceforge.net/>
  - <http://linux-hotplug.sourceforge.net/> (look for fxload)
- optionally [SDCC](http://sdcc.sourceforge.net/) to compile the firmware yourself

## What you've got to do ##
1. Install libusb.
2. Compile the host application:  
  `cd app`  
  `make all`  
3. Install your preferred firmware downloader.
4. Build the multiboot cable.
5. Connect the GBA and the proto board with the cable.  
   Power on the GBA.
6. Download multiboot.ihx to the proto board.
7. Run the host application:  
   `xmb`  
   or  
   `mb <filename>`  

Once the firmware has been downloaded to the proto board, consecutive GBA files can be transfered to the GBA. If communications hang, unplug the proto board and download the firmware again.

## Where to find additional information ##
- <http://program.at/Andrew/>
- <http://ajo.thinknerd.com/gba/>
- <http://davidwu_2001.tripod.com/root/>
