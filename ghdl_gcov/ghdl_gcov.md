---
title: Evaluating VHDL Code Coverage Using GHDL and gcov
---

# Evaluating VHDL Code Coverage Using GHDL and gcov #

## Introduction ##
GHDL is a frontend for gcc that compiles programs/designs written in VHDL into executable files. It enables execution of such designs, rendering a dedicated simulator obsolete that is usually necessary to execute pre-compiled VHDL code. Like the underlying gcc compiler, GHDL is open-source and is therefore free from restrictions and license issues that arise with commercial simulators.
Being integrated into the gcc compiler environment, GHDL itself and the resulting executables include features that are developed for software debugging but can be beneficial for the hardware design and debug process as well. One of these features is the measuring code coverage with gcov, the GNU coverage testing tool. This document intends to show how gcov can be used in conjunction with GHDL.

Further information can be found at

- <http://ghdl.free.fr/>
- <http://gcc.gnu.org/>

## Motivation ##
Verification of a hardware design is an important and time consuming job. Personally, I think that verifying a design is more effort than coding it - and you never know when you're finished. The decision whether a design has undergone enough verification or not is not easy and there are no signs that tell you that you're 100% done.

It's easier to figure out that you're not completely done. For example when the verification tests have not covered all possible states of the design and certain functionality is still untested. This task can be performed automatically with tools looking how the test case(s) stimulate the design and protocolling all actions within the design. This information is presented in a way that enables the designer to determine whether there are coverage holes in the tests. He can then add tests that fill these holes and complete the coverage of his tests.

Several metrics have been introduced to measure code coverage. To name but a few:

- **Line coverage**  
  Tells you how often a line of the source code has been executed.
- **Branch coverage**  
  Gives information how often each branch was taken.
- **Toggle coverage**  
  Show the number of state changes on all signals, sometimes in relation to other signals.
- **Condition coverage**  
  Identifies the combinations of all elements within a conditional expression that were observed.

This document will focus on line coverage as it is the most simple metric and will touch branch coverage.

Keep in mind that 100% code coverage does not mean 100% verification. Code coverage does not mean test coverage, i.e. a program that stimulates all code lines but does not check anything achieves 100% code coverage but almost no test coverage (and is thus useless for verification). As was written before: code coverage tells you where your verification is definitley missing something. It does not tell you anything about its quality.

## Prerequisites ##
In addition to a recent version of GHDL (0.11.1 at the time of this writing) you need the gcc compiler suite. It has to be the same gcc version that has been used to compile GHDL. For GHDL 0.11.1 this is gcc 3.3.3 from <http://gcc.gnu.org/>. Some linux distributions enhance the precompiled gcc binaries with additional patches, but these influence the behaviour and file formats of gcov. Therefore, it's recommended to use the original gcc.

It took me quite a while to figure this out, but I didn't want to migrate from the version I am currently using. To accomodate this requirement, gcc can be installed into a separate place where it does not interfere with other versions of gcc. All it takes to set it up is this:

```
$ tar -xjf gcc-core-3.3.3.tar.bz2
$ cd gcc-3.3.3
$ ./configure --prefix=<some directory far away from $PATH>
$ make
$ make install
```

Before using GHDL, prepend the `bin` directory of this installation at the beginning of `$PATH`. This will eclipse the standard gcc version. The result should be:

```
$ gcc -v
gcc version 3.3.3
$ gcov -v
gcov (GCC) 3.3.3
```

## Generating Coverage Results ##
GHDL/gcc can generate line and branch coverage information for every single VHDL file of the design. GHDL is instructed to instrument the executable code with two command line options that are passed to gcc. Add the following switches to the analyze step:

`$ ghdl -a -Wc,-ftest-coverage -Wc,-fprofile-arcs ... <unit>.vhd`

This generates two additional files names `unit.bb` and `unit.bbg`. Refer to gcov(1) for contents and structure of these files.

Normally, the elaboration step needs no additional measures. But in case there are linker errors about unresolved references, they should be fixed with the following option:

`$ ghdl -e -Wl,-lgcov <toplevel configuration>`

When the design is executed, the coverage information is collected and dumped to `unit.da`. Such a file is generated for each design unit that has been analyzed with the -ftest-coverage and -fprofile-arcs switches. It contains the accumulated number of times each arc in the source code has been traversed.

Using the three files `unit.bb`, `unit.bbg` and `unit.da`, `'gcov unit.vhd'` generates the coverage report unit.vhd.gcov, listing every line of the source file annotated with the number of times each line was passed during execution. As written before, this information is accumulative because every run adds its own coverage numbers to `unit.da` instead of overwriting it and filling it from scratch. It is therefore possible to run a whole suite of tests and find the complete result of all testcases combined in this file.

A sample coverage report can be found here: [alu.vhd.gcov.gz](alu.vhd.gcov.gz). It is taken from the [T48 µController](http://opencores.org/project,t48) project at [OpenCores.org](http://www.opencores.org/).


In addition to line coverage, gcov can generate information about branch probability when the -b option is specified on the gcov command line. It takes quite a lot of guesswork to interpret the report as the notation of calls and branches used by gcov is not very intuitive in relation to VHDL.

## Conclusion ##

This report gives an introduction how to start with code coverage using GHDL, a free VHDL compiler based on gcc. Thanks to the integration with gcc, GHDL is able to produce suitable information for analyzing line coverage of arbitrary VHDL designs.Furthermore, it is expected that GHDL will benefit from future improvements of gcc/gcov let alone features of gcc that haven't been explored yet in the context of VHDL and hardware design.
