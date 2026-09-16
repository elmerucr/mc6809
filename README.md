# mc6809

![mc6809](./docs/MC6809P.jpg)

## Introduction

A library written in C++ that emulates the mc6809 cpu. The enclosed ```CMakeLists.txt``` file (standard cmake procedure) will build the library and a small test application. To use this library in your project, copy the five source files from ```./src/``` into your source tree.

The following opcode is not yet implemented:
* CWAI

A feature from the Hitachi 6309 that might be implemented (with a switch in the source code):
* illegal opcode exception (vector at $fff0)

At this point it's unclear to me how many cycles the NMI, FIRQ and IRQ exceptions consume. I made a guess for 19, 10 and 19 cycles respectively.

To reduce the CPU load on the host system when the the 6809 cpu is idle (after a SYNC or CWAI instruction), 50 cycles are taken for each call to:

```cpp
uint16_t mc6809::execute()
```

Feel free to change following lines in ```mc6809.hpp```:

```cpp
#define SYNC_CYCLES	50
#define CWAI_CYCLES	50
```

## API

### Constructor

```cpp
mc6809::mc6809()
```

### Read and Write to Memory

Abstract class, read8 and write8 must be implemented in your subclass.

Make sure the connected memory has a functioning ROM and vector table from ```$fff0``` to ```$ffff```. Please note that an extra vector at ```$fff0``` (originally reserved by Motorola) has been added that enables handling of illegal opcodes (a feature borrowed from the Hitachi 6309).

### NMI / FIRQ / IRQ

When nothing is assigned by the hosting software, the pin states will default to high (```1``` or ```true```) internally, effectively meaning no exceptions of the above three types will happen. It is up to the programmer to supply proper connections:

```cpp
void mc6809::assign_nmi_line(bool *line)
```

```cpp
void mc6809::assign_firq_line(bool *line)
```

```cpp
void mc6809::assign_irq_line(bool *line)
```

These functions take a pointer to a boolean value (the actual line/value that represent interrupt states from the connected devices). If more devices are to be connected to one line, this must be separately programmed (see ```exceptions_ic``` class in [lime](https://github.com/elmerucr/lime) source code for an example).

### Reset

```cpp
void mc6809::reset()
```

### Execute

```cpp
uint16_t mc6809::execute()
```

Main execute function. Runs only one instruction, and returns the number of cycles consumed. Checking for breakpoints must be done with the ```mc6809::breakpoint()``` member function inbetween calls to the ```mc6809::execute()``` function. In the broadest sense, one instruction also means starting an exception (be it nmi/firq/irq).

Running more that one instruction or the ability to run ```N``` cycles has been considered but is not implemented. In most use cases we want to do nmi/firq/irq things immediately after each instruction anyway. Also checking for breakpoints becomes simpler this way.

## Assemblers and Compilers

* [asm6809](https://www.6809.org.uk/asm6809/) - A portable cross assembler targeting the Motorola 6809 and Hitachi 6309 by Ciaran Anscomb.
* [asmx](http://xi6.com/projects/asmx/) - A multi-CPU macro assembler by Bruce Tomlin for many major 8-bit and 16-bit CPUs, which can assemble code for multiple CPUs in the same source file.
* [CMOC](https://freshcode.club/projects/cmoc) - CMOC is a 6809-generating cross-compiler for a large subset of the C language by sarrazip.
* [LWTOOLS](http://www.lwtools.ca) - LWTOOLS is a set of cross-development tools for the Motorola 6809 and Hitachi 6309 microprocessors by Willam 'Lost Wizard' Astle.

## Links

* [lime](https://github.com/elmerucr/lime) - A virtual computer system that draws inspiration from computing platforms such as the Commodore 64, the Atari ST and the Nintendo Gameboy.
* [Moira](https://github.com/dirkwhoffmann/Moira) - Motorola 68000 cpu emulator written in C++ by Dirk W. Hoffmann.
* [vAmiga](https://dirkwhoffmann.github.io/vAmiga/) - An Amiga 500, 1000, or 2000 on your Apple Macintosh by Dirk W. Hoffmann.
* [vasm](http://sun.hasenbraten.de/vasm/) - A portable and retargetable assembler by Volker Barthelmann.
* [VICE](http://vice-emu.sourceforge.net) - The Versatile Commodore Emulator.
* [VirtualC64](https://dirkwhoffmann.github.io/virtualc64/) - A Commodore 64 on your Apple Macintosh by Dirk W. Hoffmann.
* [vlink](http://sun.hasenbraten.de/vlink/) - A versatile linker by Frank Wille.
* [vrEmu6502](https://github.com/visrealm/vrEmu6502) - 6502/65C02 emulator library (C99).

## References

Leventhal, Lance A. 1981. 6809 ASSEMBLY LANGUAGE PROGRAMMING. OSBORNE/McGraw-Hill.

Motorola. 1981. MC6809-MC6809E 8-BIT Microprocessor Programming Manual. Motorola Inc.

Motorola Semiconductors. 1983. MC6809 Datasheet.

Osborne, Adam. 1976. An introduction to microcomputers - Volume I Basic Concepts. SYBEX.

Zaks, Rodnay and William Labiak. 1982. Programming the 6809. SYBEX.
