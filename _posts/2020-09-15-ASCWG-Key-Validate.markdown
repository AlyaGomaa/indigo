---
title: "[Writeup] ASCWG2020 - Key Validate"
layout: post
date: 2020-09-15 2:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- CTF
- Writeup
- Reverse
category: blog
author: coreflood
description: ----
---
# Challenge
Download the binary [here](https://filebin.net/8izcu5w48xia6rk2)

# Solution

At first the binary does some weird computations which turned out to be the stored hashes

Then, We're asked to enter an 8 character long key

![](https://raw.githubusercontent.com/N33rdZ/Blog/gh-pages/_posts/images/Key-Validate/input.png)


then it iterates through each character of our input, MD5 hashes it, and compares it to the ```stored hashes ^ index & 0xff```

![](https://raw.githubusercontent.com/N33rdZ/Blog/gh-pages/_posts/images/Key-Validate/hashing.png)

you can safely ignore every ```+i*16``` in the decompilation as its only purpose is to get each two characters(one byte) of the stored hash.


so we're left with this:
```((stored_hashes[j]) ^ j & 255U) == hashed_char[j]```

now to get the flag we need to:
- extract the hashes from the binary
- decode each hash
- hash each printable character and compare the hash with the stored hash (bruteforce the flag)

so we set a breakpoint at 0x0040137E and extract 40 * 16 bytes at 0x5658F088 using [this](https://gist.github.com/herrcore/01762779ae4ac130d3beb02bf8e99826) plugin

![](https://raw.githubusercontent.com/N33rdZ/Blog/gh-pages/_posts/images/Key-Validate/bp.png)


here's the final script:
```python

x = "e359eda78df08665f904dd3a67686a913501d18305d0a867599b58252674408ec14635b4e10b33d04b11216ec46cc21d4224dcaf49a8f41eeaab207bd4a4d260cfe00e9e310ad78b2d1d83ac556482a8949b106bf4ec75d555950ac13328dac431efb0f64116ea8cb2adf6228a63c7bcc2848d34371307c19d4aa2a74a69fa22f2946c1ff91844ae5a1b90cb641462d000a272709167d1575b519797e4d5562ca366620f47307ea62a72ad05809d5e9bc2a45c2fc928749e6a2ba0fb542452e021a6d55401ebd330abf1c18e248c22fdab5a448900697633c26cc73c6fcebd7eaa6bea1379d251d07d8d7ad3ac66b1dcbbb2425d17a3d23a6d40eae367c03cce4153927591914009264153299c423a5293969e8203062e873616fae08bb5da65c379cd87add0a645d924fd1a47484ab1dcfafa4d7f69d4c91009b5e2ce9a84ccc236a2d2493047d2fc01333d5aaddfe22244bccf29c8947e8acb401bb4c4b200914863e5c1b5d7a9c34e9e8c10613307e0429290718731b7bbb177770435b6cc63d96d270d7006e579845dbae7e8ea115148679c1217cc8dbb9b76bfd55cef09d2b44c3fd938648e7a3bb0eb443442f0bc70c70a7444001f897a2359d5ca98deb2d42c5fb95804ee1a5bd08b24542290fd688c5dce47b16079c15f8d6edce5ec2f2cc267713083088e0e3512156a8a3542040db4b79387411dca2227cae97aa29032e2e001f741c7cbc107077445c6bc5a9b1ae38922a1208d7d8a235c96412c5234ccbf59b8e40efabb306bc4b4c27082c4cd7477534781dd0ae2e70a29ba6211cf90d66136caac928dd602aa43e79cb309bdf7dda0d635a9548d6a37383ac18caaaa1d2f3984994059e5b29ecad49c00cee9f1e6ebeee3a7cce2fea98d0098"

import hashlib
import string
from bitsbehumble import *

# split x into usable bytes
x = hexstring_to_array(x) 

#decode the hashes
XORed_hash = "" 
for j in range(len(x)):
    byte= x[j]
    xored_byte= hex(  int(byte[2:],16) ^ (j & 0xff) )[2:].zfill(2)
    XORed_hash += xored_byte

#create a list of the decoded hashes
hashes=[ XORed_hash[i:i+32] for i in range(0,len(XORed_hash),32)]

#bruteforce each chaaracter of the flag
flag=""
for i in range(len(hashes)):
    for c in string.printable: 
        if hashlib.md5(c.encode()).hexdigest() == hashes[i] :
            flag += c
            break
print(flag)

```
# Flag

![](https://raw.githubusercontent.com/N33rdZ/Blog/gh-pages/_posts/images/Key-Validate/op.png)


