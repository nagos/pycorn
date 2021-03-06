Pycorn
======

Pycorn is an interpreted operating system written in Python, intended to
recreate the style of learning available on 16-bit microcomputers. Device
drivers, file systems, network protocols can all be implemented in Python with
no C or assembler code.

The goal of the project is to create a usable interpreter-centric OS
environment with the capabilities of modern computers; allowing a smooth
learning curve from the acquisition of basic programming skills right up to
implementing new OS-level functionality and drivers. Performance is not a goal,
though it would be nice :) Current development is targeted at a variety of ARM
platforms due to the relative simplicity of the hardware.

License
-------

> This program is free software: you can redistribute it and/or modify
> it under the terms of the GNU General Public License as published by
> the Free Software Foundation, either version 3 of the License, or
> (at your option) any later version.
>
> This program is distributed in the hope that it will be useful,
> but WITHOUT ANY WARRANTY; without even the implied warranty of
> MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
> GNU General Public License for more details.
>
> You should have received a copy of the GNU General Public License
> along with this program.  If not, see <http://www.gnu.org/licenses/>.

How it all fits together
------------------------

Pycorn uses a cross-compiling gcc and the newlib embedded C library.  This
provides enough 'stuff' to compile the main python interpreter into a library.

To actually make a bootable image a linker script and entry point are required
which are implemented by pycorn. These are architecture specific and include a
few machine specific details: a small linker script with some platform physical
addresses includes the main architecture linker script. The entry point uses a
small amount of assembly then switches to position-independent C code for the
majority of platform setup, then calls a generic C library setup function
written in C.

Right now we only have them for Marvell's PXA270 system-on-chip, as used in the
Gumstix Verdex development board <http://www.gumstix.com>. This board can be
simulated by QEMU. A main() function is also required which invokes the python
interpreter with some arguments, and though this is written in C it is
portable.

Of course, it won't actually do anything visible; newlib has no actual IO
facilities. A platform-specific serial debug driver is used for now, hooked up
to newlib's stdin/stdout/stderr.

And.. that's it for now. There is no way to import any modules which aren't
built into the interpreter as all file access fails right now, and Python's
builtin readline is really stupid so you can't even backspace.. but it works!

Binary releases
---------------

Pycorn is not very useful as a binary image at this point, because the builtin
Python code is not yet finished, and so it is expected that people will want to
play around with the Python code and build a new image from it.

However, the actual native binary contains only the minimum amount of code to
start up the interpreter and begin to import stuff, and binary releases contain
a prebuilt copy of this binary. This means you don't need a compiler.

Requirements
------------

Pycorn requires several things in order to build, even if you are using a
binary release:

1.  makepp 2.0. makepp is a nice make replacement with lots of nifty features;
    used primarily because I hate make's handling of subdirectories. See
    <http://makepp.sourceforge.net/> for their project page. deb and rpm
    packages are available there. There used to be a makepp snapshot in the
    pycorn tree but this has been removed now the 2.0 release is out.

2.  If you aren't using a binary release, you need to build the arm-eabi
    crosscompiler toolchain. It needs to be built specifically for Pycorn; a
    binary package won't work. To build it, you will need the normal host
    toolchain for your system (on debian-alikes, install build-essential), the
    header files for libgmp, libmpfr and libmpc (the dev packages for them from
    your distro), and texinfo. Then, run:

        cd toolchain; makepp TOOLSPREFIX=/dir/to/install/to

    Once it's built, add the bin directory inside TOOLSPREFIX to your path.

3.  If you aren't using a binary release, install autoconf. Python's configure
    script needs regeneration after patching for cross-compilability.

Building
--------

If you are using a binary release, skip step 1.

1.  Run `makepp seeds/hello/hello.uimage`. This is a "hello world" program which
    should load into u-boot and print on the default platform serial port.

    If you have an actual Gumstix Verdex board you should be able to load this
    by serial, tftp, or MMC, and execute it with bootm.

    If you don't, you can simulate it under QEMU by running
    `makepp seeds/hello/run`, assuming you have QEMU installed. The bootloader
    will be configured automatically so it should just run. Hit C-a x
    (C-a a x under screen) to kill QEMU.

2.  Run `makepp seeds/pycorn/pycorn.uimage`. This is the python interpreter,
    with the minimum amount of code to boot pycorn built into it.

    Again, if you have an actual Verdex board it should run from u-boot.

    Otherwise, `makepp seeds/pycorn/run` to launch it in QEMU.
