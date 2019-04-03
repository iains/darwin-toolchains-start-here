# darwin-toolchains-start-here

WHAT
====

To build a competent modern Darwin toolchain several pieces are required and (depending on the system version you are starting on) different stages.

A bunch of what might start out here in the README is probably better placed in a Wiki - but for now...

The **objective**
=================

A common toolchain supporting C-family, Fortran, Ada etc. hosted on Darwin9+ (OS X 10.5+).
The toolchain will eventually include both GCC and Clang compilers..

PRE-REQUISITES
==============

* GIT (somewhat obviously)
* cmake 3.5+
* Python 2.7 (I've been using 2.7.11 on Darwin9+) - You'll need to build/obtain Sphinx for documentation support.
* IFF you want to support Ada, then you'll have to find a compiler supoorting Ada *and* c++11 that runs on your platform (e.g. the darwin-5.3r0 release).
* To make the newer versions of ld64 (or to support any SDK newer than 10.11) you need a libtapi that supports tapi-v3. There's a way to build that too.
* GCC prerequisites (GMP, MPFR, MPC and optionally ISL) these can be bootstrapped along with the compiler or prebuilt.

STARTING FROM a c++11-capable compiler (e.g. the darwin-5.3r0).
===============================================================

This is the only scenario I've tested (it might be possible to do the sequence "STARTING FROM XCODE ONLY" for 8.3r0 without going via 5.3 but it's untested).

Whatever way you go about it - you will need a version of GCC that support the emulated TLS via a crt (so darwin-5.3r0 or darwin-8.3r0).

1. LLVM base for ld64
* LLVM-7.1-WIP
* tapi-2.0.1-for-LLVM-7.1-with-emuTLS.

These source trees should be side-by-side (so that the symlink in clang/tools finds tapi).

Build and install at least LTO, llas, libtapi, libtapi-headers, dsymutil, llvm-symbolizer (I usually also build llvm-dwarfdump, llvm-objdump, llvm-ar, llvm-nm and llvm-strip).

2. Xtools
* darwin-xtools-2.2.3

Build and install this.

3. GCC
* darwin-gcc-8.3r0
you can configure this with --with-as=/path/to/llas and --with-ld=/path/to/new-ld

It's probably best to bootstrap this, since you're likely starting from GCC 5 or something like that.

If you want to do a "full bootstrap" including the LLVM base and the Xtools, then repeat 1-3 using the last built 8.3 compiler as the bootstrap compiler each time (in that case, you might as well --disable-bootstrap to save time).

... more detail to follow, maybe.

STARTING FROM XCODE ONLY (or some other Ada than the one above).
===============================================================

powerpc,i686-apple-darwin9 (OS X 10.5)
--------------------------------------
XCode 3.1.4
set up your installation to default to gcc-4.2/g++-4.2 and/or to point to the Ada-capable compiler you're using.

i686,x86_64-apple-darwin10 (OS X 10.6)
-------------------------------------
XCode 3.2.6
(or point to the Ada compiler.. yada yada)

x86_64-apple-darwin11 (OS X 10.7)
---------------------
XCode 4.6.3

x86_64-apple-darwin12 (OS X 10.8)
---------------------
XCode 5.1.1

x86_64-apple-darwin13 (OS X 10.9)
---------------------
XCode 6.2

x86_64-apple-darwin14+ (OS X 10.10+)
----------------------
XCode 7.2+

SEQUENCE FOR DARWIN9-DARWIN13
=============================

 - you'll need to build gmp, mpfr and mpc as a minimum to support these phases. I recommend building them as static libs.
 - I've been using 6.1, 3.1.3 ans 1.0.3 respectively.
 - It's a good idea to make the compiler triple explicit (with --target= --host= --build= on the configure line). So, for example, --target=x86_64-apple-darwin10 .. etc.  You'll need to know the triple to install symlinks so that the new cctools/ld64 get found.

Stage0
------
 1. Bootstrap GCC-5.3 (the one from here) with the system compiler (or your Ada one)
 2. Build darwin-xtools with that compiler (there are a bunch of warnings, when using earlier XCode, these are expected)
 3. Install darwin-xtools into the same dir as the compiler, and make sure that there's a symlinked /install/path/compiler-triple/bin that points to ../bin (so that the new as, ld, etc are found instead of the system ones)

... that get's us to a starting point where we have a base c++11 compiler, but we don't have LLVM LTO support in the tools.

Stage1 - Rebuild with the new xtools
------------------------------------
We don't need a bootstrap here, we're building with the just-made compiler.

point your PATH to the just-made compiler.

1. Build darwin-xtools with that, and install (the linker warnings about line numbers should be gone).
2. Build GCC-5.3 (but --disable-bootstrap, to make things quicker).
3. Again make sure that there's a symlink /install/path/compiler-triple/bin => ../bin

Stage2 - Build A toolchain supporting LLVM LTO (and GCC LTO, of course).
------------------------------------------------------------------------
**NOTE** we won't get to **use** the LLVM LTO for some time yet (we'd need to build clang etc.)

point your PATH to the just-made compiler.

1. Build llvm-only [we only want libLTO.dylib, llvm-symbolizer and, in due course, llvm-dsymutil] - use the darwin-llvm branch from the llvm-project here, to get improved PowerPC support.  Last tested with a branch based on svn 256450.
2. Install LTO, llvm-symbolizer, llvm-dsymutil
3. build darwin-xtools with XTOOLS_LTO_PATH pointing to the LTO installation.
4. bootstrap GCC-5.3.
5.  Again make sure that there's a symlink /install/path/compiler-triple/bin => ../bin

Now you should have a toolchain with ld64 --version claiming LTO support.

We can then move on to building more of LLVM.....

SEQUENCE FOR DARWIN14+
======================

You can just bootstrap GCC-5.3 with the XCode 7.2 tools (they are newer than darwin-xtools).
However, IFF you want to have a toolchain like the ones for earlier Darwin, then start at stage1.





