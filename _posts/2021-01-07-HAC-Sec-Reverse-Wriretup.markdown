---
title: "[Writeup] HAC-Sec CTF - Reverse Engineering "
layout: post
date: 2021-01-07 10:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- CTF
- Reverse
- Writeup
category: blog
author: coreflood
description: "HAC-Sec CTF Reversing Writeup"
---

- [Side B](#side-b)
- [Math Head](#math-head)
- [xchg](#xchg)
- [Bibliotheque](#bibliotheque)


# Side-B
![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/HAC-Sec-CTF-Writeups/sideB.png)

The challenge consists of 2 files, Challenge and encrypted_flag.txt

When we run the file we get
![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/HAC-Sec-CTF-Writeups/1.png)

So now we know the flag is probably AES encrypted and we need to find the key and IV to decrypt it.

opening the file in ida we get
![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/HAC-Sec-CTF-Writeups/2.png)


first we need to pass the anti debugging function ptrace using a debugger to get to the function that downloads the image
so we set a breakpoint at the ptrace call and when it's hit we change the eip to point to ```download_img```

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/HAC-Sec-CTF-Writeups/3.png)


after the ```download_img``` function is done we see an image dropped to disk


<img src="https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/HAC-Sec-CTF-Writeups/Side-B.jpg" width="230"/>

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/HAC-Sec-CTF-Writeups/4.png)


what a cool author ha :blush:

after it's downloaded, it reads 16 bytes from offset 533 and store it as the key

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/HAC-Sec-CTF-Writeups/5.png)

now that we have the key we need the IV to get the flag

to get the IV the program opens itself, reads 16 bytes from offset 1456 and xors each byte with 0x11

to get the iv using the debugger we need to bypass the ```__debugbreak()``` which is another anti-debug technique 

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/HAC-Sec-CTF-Writeups/6.png)

so we set the eip at seekg() and continue executing until v7 is populated with the IV and then we extract it

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/HAC-Sec-CTF-Writeups/7.png)

now that we have the key and the iv we can decrypt the flag

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/HAC-Sec-CTF-Writeups/8.png)

## Flag
HAC{mu$1c_t0_b3_murd3r3d_by_b4by!;)}

---

# xchg
Difficulty: Medium 300

challenge.asm:
```
section .text:
global _start
_start:
    lea edx, s    
    mov di,edx   
    lea bh, flag   
    mov bl , bh 
    mov cl , 1fh
    add bl , cl  

    L1 : 
        add di,1  
        mov al , [edx]    
        xchg [bh] , al   
        mov  ch ,[di]       
        xchg  [bl] , ch    
        dec bl
        add edx,2
        inc bh
        cmp edx , cl
        jl  L1
        ret
section .data:
s db 'H', '}', 'A', '!', 'C', 'g', '{', 'n', 't', '1', 'h', 'n', '1', 'n', 's', '1', '_', 'g', '1', '3', 's', 'b', '_', '_', 'j', '3', 'u', 'h', '$', 't', 't', '_' 
flag db ''
```

The idea of the challenge is to translate assembly to a high level language and get the flag

all the it does is get 2 letters of the data variable and place the first of them at the beginning of the flag and the second at the end of the flag. it keeps doing this recursively until we get the flag 

So here's the python script:

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/HAC-Sec-CTF-Writeups/11.png)

## Flag:  

HAC{th1s_1s_ju$t_th3_b3g1nn1ng!}

---

# Math Head

Difficulty: Medium 250
Description: all flag characters range from 44 to 126.

When we run the challenge we get this:

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/HAC-Sec-CTF-Writeups/1-.png)

so it needs a key and it will validate it
when we open the challenge in ida we see this

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/HAC-Sec-CTF-Writeups/2-.png)

so the key length must be 24 and separated by ‘-’

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/HAC-Sec-CTF-Writeups/3-.png)

23 equations and 23 variables and we need to solve them all to get the flag
so we use z3 and give it the equations and the constraints in the description and it’ll do the magic
here’s the script

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/HAC-Sec-CTF-Writeups/4-.png)

the script outputs the key , we give it to the binary and it gives us the flag.

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/HAC-Sec-CTF-Writeups/5-.png)


## Flag
HAC{34zy-p34zy-l3m0n-squ33zy}


---

# Bibliotheque
Challenge difficulty: Easy 100

We have the libfoo.so library 
When we open it with ida we see two interesting functions 
```decode_flag``` and ```print_flag```

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/HAC-Sec-CTF-Writeups/s1.png)

so print_flag only calls decode_flag 
here's decode_flag:

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/HAC-Sec-CTF-Writeups/s2.png)

it simply concatenates a string and sends it to sub_B16 that does the actual base32decoding

so the solution would be so simply call the ```print_flag``` function from the library using a python script:

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/HAC-Sec-CTF-Writeups/s3.png)

2- or do it the hard way and get the encoded base32 string  "JBAUG6ZENA2HEM3EL5WDCYTSGRZDCM3TL42HEM27MZKW4IL5" from ```decode_flag``` function , reverse ```sub_B16``` and figure out it does base32 decoding and decode that string using any website or a python script

# Flag
HAC{$h4r3d_l1br4r13s_4r3_fUn!}




