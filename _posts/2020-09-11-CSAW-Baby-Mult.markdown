---
title: "[Writeup] CSAW2020 - Baby Mult "
layout: post
date: 2020-09-11 02:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- CTF
- Reverse
- Writeup
category: blog
author: coreflood
description: ~.~
---
# Challenge
We're given a [program.txt](https://github.com/AlyaGomaa/blog/blob/gh-pages/_posts/Baby_Mult/program.txt)

# Solution
We're given some values that range from 0 to 255 and the file is called program.txt so it's probably shellcode

so we convert these values to hex and store them in a file any hex editor.
![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/Baby_Mult/binn.png)


now we need to convert it to readable disassembly so i ran ``` ndisasm -b32 x ``` but the instructions didn't make any sense so i tried ``` ndisasm -b64 x ``` to view x64 assmebly instead of x86

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/Baby_Mult/inst.png)

now the instructions are pretty straightforward since they're mainly imul and not idiv :'D, the program calculates each part of the flag by multiplying two or more values and stores them in a local variable.

you can calculate the flag using a python script or you can compile the binary and let gdb do the work for you

here's [x.asm](https://github.com/AlyaGomaa/blog/blob/gh-pages/_posts/Baby_Mult/x.asm)

## Compiling
``` nasm -f elf64 x.asm ``` to assemble the program,  
``` gcc x.o -nostdlib -o x ``` to compile the object file nasm produced into an executable file, *-nostdlib* because gcc will link the standard C libraries by default which already contain a *_start* that invokes main entry point and then complain that it has multiple entry points:'D, 
``` chmod +x x ``` to make it executable.


## Debugging:
``` gdb -q x -ex "start" -ex "b *0x000055555555431d" -ex "c" ``` to let the program finish execution

``` x/s $rbp-0x80  ```

``` x/s $rbp-0x88 ```

``` x/s $rbp-0x90 ``` 

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/Baby_Mult/flagg.png)


# Flag

``` flag{sup3r_v4l1d_pr0gr4m} ```
