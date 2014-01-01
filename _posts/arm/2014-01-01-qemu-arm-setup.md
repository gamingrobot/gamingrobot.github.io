---
layout: post
title: Setup Qemu for ARM on Wheezy
tags : [qemu, arm, debian, wheezy]
---
{% include JB/setup %}

This is a quick little blog post for how to install Qemu for ARM on Debian 7.0(Wheezy).

## Installing Toolchains

Edit /etc/apt/sources.list and add
    
    deb http://emdebian.org/debian/ stable main
    deb http://ftp.us.debian.org/debian/ squeeze main

The sqeeze repo is for the package `libgmp3c2` that is unavailable on wheezy.

Install the emdebian public key

    # apt-get install emdebian-archive-keyring
    # apt-get update

Install ARM toolchains

    # apt-get install linux-libc-dev-armel-cross libc6-armel-cross libc6-dev-armel-cross binutils-arm-linux-gnueabi gcc-4.4-arm-linux-gnueabi g++-4.4-arm-linux-gnueabi

---
## Installing Qemu

Install Qemu

    # apt-get install qemu qemu-user-static

Optional: Install Qemu GUI

    # apt-get install aqemu

---
## Hello ARM

Time to test everything is working.

`helloarm.s`

    .text
 
    .globl _start
    _start:
            mov   r0, #5         @ Load register r0 with the value 5
            mov   r1, #4         @ Load register r1 with the value 4
            add   r2, r1, r0     @ Add r0 and r1 and store in r2

    stop:   b stop               @ Infinite loop to stop execution

### Assemble, Link, Build binary

    $ arm-linux-gnueabi-as -o helloarm.o helloarm.s
    $ arm-linux-gnueabi-ld -o helloarm.elf helloarm.o
    $ arm-linux-gnueabi-objcopy -O binary helloarm.elf helloarm.bin

### Setup flash.bin

    $ dd if=/dev/zero of=flash.bin bs=4096 count=4096
    $ dd if=helloarm.bin of=flash.bin bs=4096 conv=notrunc

### Launch Qemu

    $ qemu-system-arm -M connex -pflash flash.bin -nographic -serial /dev/null

There should now be a `(qemu)` prompt, type `info registers` and should see something like this

    R00=00000005 R01=00000004 R02=00000009 R03=00000000
    R04=00000000 R05=00000000 R06=00000000 R07=00000000
    R08=00000000 R09=00000000 R10=00000000 R11=00000000
    R12=00000000 R13=00000000 R14=00000000 R15=0000000c
    PSR=400001d3 -Z-- A svc32

`R02=00000009` Register 2 has the value 9 in it so everything is working.

A more in-depth explanation of this helloarm can be found [here](http://www.bravegnu.org/gnu-eprog/hello-arm.html).



    