---
title: Merging BRAM Data Into Bitstreams
---

# Merging BRAM Data Into Bitstreams #

## Introduction ##
This text is a short discussion of a way to merge BRAM initialization data into bitstreams for Xilinx FPGAs. My intention was to avoid the lengthy resynthesis plus P&R of whole designs when the software code inside a BRAM has changed.

Browsing through the Xilinx flow docs revealed a tool called **data2mem** which promises to take BRAM initialization data and merge this into a previously generated .bit configuration file. Unfortunately, the available information is quite brief so a couple of trial-and-error iterations had to be run through until I got everything working as expected. Following you'll find some notes about what I found out so far.

## Starting Point ##
Everything below was done with Xilinx WebPACK 8.1-2, torturing a Spartan 3 device (xc3s1000-4-fg456 to be precise). The design contains a Z80 core ([T80](http://www.opencores.org/project,t80) from [opencores.org](http://www.opencores.org/)) which fetches its code from a 4 k ROM located at address 0x0000 on the processor bus. For developing code, [SDCC](http://sdcc.sourceforge.net/) is in place while [srecord](http://srecord.sourceforge.net/) is used to postprocess the binary and hex files.

It has proven necessary to partition the 4 k ROM into two RAMB16_S9 primitives instantiated directly in the VHDL design hierarchy. Assigning them unambiguous instance names - here they will be called rom_low_b and rom_high_b. I didn't take measures to initialize them directly in the VHDL code. Just for the reason to have a clean and empty configuration bit-file as a starting point.

## Preparing data2mem ##
Basically, one requires two inputs for data2mem:

1. Initialization code in "MEM" format.
2. A so called "BMM" file describing the internal memory layout.

First, I thought that converting my binaries or hex-files to this MEM format might be the biggest obstacle. But soon I discovered that MEM is simply the Verilog VMEM format, commonly used in the Verilog world of designing. Glad to see that [srecord](http://srecord.sourceforge.net/) supports this format for reading and writing. So all that is needed fits in one single command line:

`$ srec_cat rom_code.bin -binary -o rom_code.mem -vmem 8`

It's maybe worth noting that `rom_code.bin` has been filled to the full 4 k using srec_cat's -fill option.


Generating the BMM file resulted in a lot more guesswork because I could not find an exhaustive documentation for these files. Starting from `data2mem -h`, I experimented with the -mf option until the tool produced something that looked reasonable.

```
$ data2mem -mf p z80map PPC405 0            \
               a rom_code b 0x0000 8        \
               s RAMB16 0x0800 8 rom_low_b  \
               s RAMB16 0x0800 8 rom_high_b \
           -o p rom_code.bmm
```

The result is

```
// BMM LOC annotation file.
//
// Release 8.1.02i - Data2MEM I.27, build 1.4.5 Jan 11, 2006
// Copyright (c) 1995-2006 Xilinx, Inc.  All rights reserved.
//
// Created on 08/21/06 01:26 am
//


///////////////////////////////////////////////////////////////////////////////
//
// Processor 'z80map', ID 0, memory map.
//
///////////////////////////////////////////////////////////////////////////////

ADDRESS_MAP z80map PPC405 0


    ///////////////////////////////////////////////////////////////////////////////
    //
    // Processor 'z80map' address space 'rom_code' 0x00000000:0x00000FFF (4 KB).
    //
    ///////////////////////////////////////////////////////////////////////////////

    ADDRESS_SPACE rom_code COMBINED [0x00000000:0x00000FFF]

        ///////////////////////////////////////////////////////////////////////////////
        //
        // Address range 0x00000000:0x000007FF (2 KB).
        //
        ///////////////////////////////////////////////////////////////////////////////

        ADDRESS_RANGE RAMB16
            BUS_BLOCK
                rom_low_b0 [7:0];
            END_BUS_BLOCK;
        END_ADDRESS_RANGE;


        ///////////////////////////////////////////////////////////////////////////////
        //
        // Address range 0x00000800:0x00000FFF (2 KB).
        //
        ///////////////////////////////////////////////////////////////////////////////

        ADDRESS_RANGE RAMB16
            BUS_BLOCK
                rom_high_b0 [7:0];
            END_BUS_BLOCK;
        END_ADDRESS_RANGE;
    END_ADDRESS_SPACE;

END_ADDRESS_MAP;
```

Data2mem appends a 0 to the instance names which has to be removed before adding the file to the ISE project. Bitgen is then supposed to annotate the location of the two BRAMs to this file. In fact, bitgen copies the contents of this file to `rom_code_bd.bmm` and inserts `PLACED = X1Y6` attributes to the `rom_low_b` lines. Unfortunately, bitgen annotates only the first instance and skips `rom_high_b` for some reason.

After fiddeling with the file, I could find a combination that finally works. Just merge the two separate `ADDRESS_RANGE` definitions into a single one:

```
// BMM LOC annotation file.
//
// Release 8.1.02i - Data2MEM I.27, build 1.4.5 Jan 11, 2006
// Copyright (c) 1995-2006 Xilinx, Inc.  All rights reserved.
//
// Created on 08/21/06 01:26 am
//


///////////////////////////////////////////////////////////////////////////////
//
// Processor 'z80map', ID 0, memory map.
//
///////////////////////////////////////////////////////////////////////////////

ADDRESS_MAP z80map PPC405 0


    ///////////////////////////////////////////////////////////////////////////////
    //
    // Processor 'z80map' address space 'rom_code' 0x00000000:0x00000FFF (4 KB).
    //
    ///////////////////////////////////////////////////////////////////////////////

    ADDRESS_SPACE rom_code COMBINED [0x00000000:0x00000FFF]

        ///////////////////////////////////////////////////////////////////////////////
        //
        // Address range 0x00000000:0x000007FF (2 KB).
        //
        ///////////////////////////////////////////////////////////////////////////////

        ADDRESS_RANGE RAMB16
            BUS_BLOCK
                rom_low_b [7:0];
            END_BUS_BLOCK;


        ///////////////////////////////////////////////////////////////////////////////
        //
        // Address range 0x00000800:0x00000FFF (2 KB).
        //
        ///////////////////////////////////////////////////////////////////////////////

            BUS_BLOCK
                rom_high_b [7:0];
            END_BUS_BLOCK;
        END_ADDRESS_RANGE;
    END_ADDRESS_SPACE;

END_ADDRESS_MAP;
```

Now bitgen assigns the locations properly to each of the RAMB16 instances:

```
rom_low_b [7:0] PLACED = X1Y6;
...
rom_high_b [7:0] PLACED = X1Y7;
```

## Running data2mem ##
As written above, bitgen generates an annotated BMM file named `rom_code_bd.bmm` that will be used for the further steps with data2mem. First, one can inspect the initial bit-file and check the configuration contents of the BRAMs in there:

`$ data2mem -bm rom_code_bd.bmm -bt design.bit -d > design.txt`

The BRAMs identified by `rom_code_bd.bmm` will be marked inside `design.txt` so it's easy to see their contents is all 0 for now.

Finally merging rom_code.mem into the bit-file is a pice of cake:

`$ data2mem -bm rom_code_bd.bmm -bt design.bit -bd rom_code.mem -o b final.bit`

A quick check with `data2mem -d` on `final.bit` reveals that the BRAMs contain the requested data. Done.

> August 2006
