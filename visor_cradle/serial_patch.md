---
title: Serial HotSync Patch for the Visor USB-Cradle
---

# Serial HotSync patch for the Visor USB-Cradle #

## DISCLAIMER ##
Before you continue reading you should be aware that all information provided here comes without any warranty. The instructions given might be wrong and therefore could cause damage to all involved material. The author is not responsible for any loss of data or any damage to your hardware. You should keep in mind that by applying the following patch you can cause severe damage to your equipmet like destroying your Computer and/or your beloved Visor. This patch worked totally for the author's device but might fail for yours!

DO NOT MODIFY YOUR HARDWARE UNLESS YOU ARE ABSOLUTELY SURE ABOUT WHAT YOU ARE DOING! YOU ARE RESPONSIBLE YOURSELF FOR ANY DAMAGE!

Any modifications might very likely void the warranty by HandSpring, Inc.

THE AUTHOR RECOMMENDS THAT YOU DO NOT MODIFY YOUR HARDWARE!

Continue at your own risk. 

## Introduction ##
When the HandSpring Visor was finally available in Germany, I could get my hands on one of the first devices sold at local shops. Unfortunately the bundled software only supports HotSync via USB under Win98. WinNT, Win95 and Win2000 are not supported yet and HandSpring is currently only spending efforts to ramp up USB-HotSync for the latter one.

The situation on the Linux side of life is not much better as USB support is not working very well within the current stable kernels. After spending several nights trying to test the latest pre-2.4.0 alpha/beta kernels all I discovered was that my motherboard came with an USB chip whose Linux-driver (OHCI) is the last one which is causing severe trouble (see [linux-usb.org](http://www.linux-usb.org/)  for the current status of Linux-USB).

A solution for all these problems became more likely when I read through the hardware docs at [HandSpring](http://www.handspring.com/). There they describe the Visor's cradle connector rather detailed and after a short time, the idea of a serial patch for my USB-cradle evolved.

## Background ##
 Although being fully compatible to the original Palm Pilot, HandSpring changed some details concerning the Visor's hardware. One of them is the cradle connector which is totally different compared to the Pilot's one. This is not suprising because the Visor offers HotSync via USB and the related signals have to be fed into the cradle somehow. Fortunately the Visor still offers a serial port at the cradle connector which allows connections to external serial devices and enables serial HotSync, too. HandSpring is selling a serial cradle as well, but it has to be ordered separately.

The serial port includes a stripped-down set of the normal  RS-232 signals by providing only the data lines TXD and RXD. Thus no hardware-handshake is possible and therefore line transmissions speeds are very limited. Nevertheless this seems to be enough for HandSpring to implement a serial HotSync protocol from which  this patch intends to benefit.

## The Gory Details ##
All what is necessary to do serial HotSyncs using the original USB-cradle is to connect the TXD/RXD signals of the Visor with the appropriate serial lines at your computer and switch the Visor to serial HotSync mode. The latter one is quite easy but the first one requires some soldering, because the signals at the Visor are using native TTL-levels and not the required levels for RS-232 interfaces. This is accomplished by this patch.

The heart of the [schematic](schematic.pdf)  is a MAX232 (level shifter) which does all the analog voltage stuff. It converts the TTL-levels from the Visor to the RS-232 levels while requirering only one single 5 V supply voltage. It is a nice circuit because it keeps all the +-9 V stuff out of our minds and lets us concentrate on the real interesting parts. The MAX232 is supplied by the computer's DTR and RTS lines which are protected by two diodes against current flow back (the circuit must not be powered with -9 V). It also needs four capacitors that are used for voltage generation. There is an optional switch after the two diodes to switch off the whole circuit when it is not used (for cautious people; might reduce the stress to the signal lines).

To finally enable serial HotSync I had to fiddle around a little bit because HandSpring does not document this topic and I have never seen an original serial cradle. The solution is to tie KBD* to GND during HotSync. You can do this permanently or you can use another switch to turn this feature off. This reverts your cradle and thus the Visor back to USB HotSync mode and also allows you to interface the Visor with external serial devices like a modem (KDB* low instructs the Visor to interpret the incoming data having a special serial protocol used e.g. with external keyboards).

That's all for the theory. In order to integrate the circuit into a cradle you can solder it on a small board that fits diagonally into the empty part of the cradle. The wires to the cradle connector can be soldered directly to the cradle connector. The board in my cradle labels pin 1 and 8 of the connector so it was easy to find the correct pins. Seen from the solder side pin 1 of the connector is at the left. Double check this with HandSpring's hardware docs before running the first trial!

The type of serial connector for your computer depends on what connector your computer has at its rear. Old motherboards provide normally a 25 pin D-SUB connector while modern ATX boards come with a 9 pin D-SUB connector. Note that there are 25 <-> 9 and 9 <-> 25 converters, so this decision does not restrict the usage to particular machines. Choose the most suitable version for your everyday work. More interesting is the gender: use a female one to directly connect it to the computer. A male one needs an additional null-modem cable but it can be used together with a normal serial cable to connect the cradle to a modem.
A last tip: the two diodes fit perfectly inside a D-SUB package, so you could solder them directly to the plug.

For further instructions on how to HotSync your Visor serially consult the documantation of the software you indent to use.

## Links ##
- [Handpsring, Inc.](http://www.handspring.com/)
- [Development Kit Manual](http://www.handspring.com/developers/developers_kit.asp)
- [Linux-USB Project](http://www.linux-usb.org/)
- [Hardware Book](http://www.hardwarebook.net/)
