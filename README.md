# curve25519
High performance implementation of elliptic curve 25519
=======================================================

Copyright (c) 2015 mehdi sotoodeh.
mehdisotoodeh@gmail.com

MIT license.


This library delivers high performance and high security while having a small
footprint with minimum resource requirements.
This library supports DH key exchange using curve25519 as well as sign/verify
operations based on twisted Edwards curve 25519.


Performance:
------------
Current version of this library sets NEW SPEED RECORDS. This is achieved 
without taking advantage of special CPU instructions or parallel processing.

The library implements a new technique (I call it FOLDING) that effectively 
reduces the number of EC point operations by a factor of 2, 4 or even more. 
The trade off is the pre-computation and cost of cached memory.
Currently 8-fold is implemented for KeyGen, Sign and DH(base point) operations 
and 4-fold for signature verification.
With 8-fold, it takes only 43K cycles on an Intel(tm) Core(tm) i7 CPU to do a 
base point scalar multiplication.

Google's implementation (http://code.google.com/p/curve25519-donna/) is used
here for performance comparison only. This library outperforms Google's code 
by a factor of 6.2 to 19.5 depending on the platform and selected language.

For best performance, use the 64-bit assembly version on AMD/Intel CPU 
architectures. The portable C code can be used for 32-bit OS's and other CPU 
types.

Note that the assembly implementation is approximately 3 times faster than C 
implementation on 64-bit platforms.
On 32-bit platforms, the biggest hit is due to usage of standard C library for
64-bit arithmetic operations. Numbers below indicate that GCC and glibc does a 
much better job than MSVC.


Timing for ed25519 sign/verify (short messages):
```
    windows7-64: VS2010 + MS Assembler, Intel(R) Core(TM) i7-2670QM CPU
        KeyGen: 44647 cycles = 13.131 usec @3.4GHz
          Sign: 48639 cycles = 14.306 usec @3.4GHz
        KeyGen: 45227 cycles = 13.302 usec @3.4GHz (Blinded)
          Sign: 49035 cycles = 14.422 usec @3.4GHz (Blinded)
        Verify: 114325 cycles = 33.625 usec @3.4GHz (Init)
                110371 cycles = 32.462 usec @3.4GHz (Check)
            
    windows7:  VS2010, Portable-C, 64-bit, Intel(R) Core(TM) i7-2670QM CPU
        KeyGen: 128777 cycles = 37.876 usec @3.4GHz
          Sign: 133823 cycles = 39.360 usec @3.4GHz
        KeyGen: 130269 cycles = 38.314 usec @3.4GHz (Blinded)
          Sign: 135011 cycles = 39.709 usec @3.4GHz (Blinded)
        Verify: 341097 cycles = 100.323 usec @3.4GHz (Init)
                307533 cycles = 90.451 usec @3.4GHz (Check)

    windows7:  VS2010, Portable-C, 32-bit, Intel(R) Core(TM) i7-2670QM CPU
        KeyGen: 542914 cycles = 159.681 usec @3.4GHz
          Sign: 556878 cycles = 163.788 usec @3.4GHz
        KeyGen: 550024 cycles = 161.772 usec @3.4GHz (Blinded)
          Sign: 563302 cycles = 165.677 usec @3.4GHz (Blinded)
        Verify: 1430160 cycles = 420.635 usec @3.4GHz (Init)
                1307354 cycles = 384.516 usec @3.4GHz (Check)

    x86_64-w64-mingw32: GNU assembler 2.25, Intel(R) Core(TM) i7-2670QM CPU
        KeyGen: 44954 cycles = 13.222 usec @3.4GHz
          Sign: 49008 cycles = 14.414 usec @3.4GHz
        KeyGen: 45628 cycles = 13.420 usec @3.4GHz (Blinded)
          Sign: 49510 cycles = 14.562 usec @3.4GHz (Blinded)
        Verify: 114642 cycles = 33.718 usec @3.4GHz (Init)
                111156 cycles = 32.693 usec @3.4GHz (Check)
                
    cygwin-32: gcc 4.5.3, Portable-C, 32-bit, Intel(R) Core(TM) i7-2670QM CPU
        KeyGen: 393512 cycles = 115.739 usec @3.4GHz
          Sign: 411468 cycles = 121.020 usec @3.4GHz
        KeyGen: 400014 cycles = 117.651 usec @3.4GHz (Blinded)
          Sign: 414946 cycles = 122.043 usec @3.4GHz (Blinded)
        Verify: 1046166 cycles = 307.696 usec @3.4GHz (Init)
                954980 cycles = 280.876 usec @3.4GHz (Check)
```

Timing for DH point multiplication:
```
    windows7-64: VS2010 + MS Assembler, Intel(R) Core(TM) i7-2670QM CPU
        Donna: 779653 cycles = 229.310 usec @3.4GHz -- ratio: 18.035
        Mehdi: 43229 cycles = 12.714 usec @3.4GHz -- delta: 94.46%      ** MSASM **

    Mingw-x86_64: GNU assembler (GNU Binutils) 2.25
        Donna: 851314 cycles = 250.386 usec @3.4GHz -- ratio: 19.590
        Mehdi: 43456 cycles = 12.781 usec @3.4GHz -- delta: 94.90%      ** GNU ASM **
    
    windows7:  VS2010, Portable-C, 64-bit
        Donna: 779753 cycles = 229.339 usec @3.4GHz -- ratio: 6.200
        Mehdi: 125761 cycles = 36.989 usec @3.4GHz -- delta: 83.87%
            
    windows7:  VS2010, Portable-C, 32-bit, Intel(R) Core(TM) i7-2670QM CPU
        Donna: 7289134 cycles = 2143.863 usec @3.4GHz -- ratio: 13.527
        Mehdi: 538846 cycles = 158.484 usec @3.4GHz -- delta: 92.61%

    cygwin-32: gcc 4.5.3, Portable-C, 32-bit, Intel(R) Core(TM) i7-2670QM CPU
        Donna: 2551492 cycles = 750.439 usec @3.4GHz -- ratio: 6.602
        Mehdi: 386498 cycles = 113.676 usec @3.4GHz -- delta: 84.85%
```


Side Channel Security:
----------------------
This library uses multiple measures with the gaol of eliminating leakage of secret 
keys during cryptographic operations. Constant-time is one of these measures and 
is implemented for private keys when they are directly operated on (no conditional 
operation based on key values). 
The second and more effective measure that this library uses is blinding. Blinding
hides the private keys by combining them with a random value.

This is a fact that constant-time implementation does not necessarily translate to
constant-power-consumption, constant-electro-magnetic-radiation and so on. It also
depends on how the underlying hardware manipulates different circuitry for each
operation. For example, a hardware multiplier may use the primitive technique of
shift-and-conditional-add or it may use barrel shifter when multiplying a power of 
2 number.

Blinding is the more effective measure with less performance penalty.
Constant-time alone, pushes attackers to dig deeper for clues.


Building:
---------
The design uses a configurable switch that defines the byte order of the
target CPU. In default mode it uses Little-endian byte order. You need to
change this configuration for Big-endian targets by setting ECP_BIG_ENDIAN
switch (see Rules.mk file on project root).

Define USE_ASM_LIB configuration when building to utilize ASM version of the library.

For building the library using the assembly sources, two assemblers are currently
supported: Microsoft Assembler (Windows) and GNU Assembler (Windows/Linux). 

- For Windows platforms, open windows/EC25519.sln using Visual Studio 2010
  and build Asm64Test project for x64 configuration.
  You also have the option of using Mingw and GNU assembler on windows.
- For Linux platforms, Debian and Ubuntu have been tested so far. For X86_64 
  assembly run: 'make clean asm' from project root. 

A custom tool creates a random blinder on every new build. This blinder is static
and will be part of the library. This blinder is only used for blinding the point 
multiplication when creating blinding context via ed25519_Blinding_Init() API.
Make sure you run the custom tool as part of your regular build.

