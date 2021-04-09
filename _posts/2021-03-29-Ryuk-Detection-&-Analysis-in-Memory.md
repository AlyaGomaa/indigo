---
title: "Ryuk Detection & Analysis in Memory"
layout: post
date: 2021-03-29 10:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Forensics
category: blog
author: coreflood
description: ~.~
---

### Lab Setup:
- windows7 64bit with 2 GB of RAM
- virtualbox
- volatility2
- Sample MD5: 3c08d1e5233c623bfc854879173544de

---

### Investgating

Using volatility i detected the profile which in our case is ```Win7SP1x64```

using ```pstree``` we can see the sample opened 4 processes one of them is ```icacls.exe```


![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/ryuk/1.png)


### Privileges

``` vol -f test.raw --profile=Win7SP1x64 privs -p 2344,2332,2488,2424,1380, | grep Enabled ``` to check enabled priveledges

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/ryuk/2.png)

nothing interesting here since the only priveledge is enabled by default for all of them 

### Process 2344 xxx.exe

#### files

``` vol -f test.raw --profile=Win7SP1x64 handles -p 2344 -t file```

nothing interesting was found here

#### Mutants

```vol -f test.raw --profile=Win7SP1x64 handles -p 2344 -t mutant```

shows that it creates 5 mutexes , google says they're malicious, found in ```Gozi Trojan `` 

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/ryuk/3.png)

####  keys

```vol -f test.raw --profile=Win7SP1x64 handles -p 2344 -t key```

shows that it accesses the network provider using the key 

```MACHINE\SYSTEM\CONTROLSET001\CONTROL\NETWORKPROVIDER\HWORDER```

this is where the user specifies the default program to open files with a specific extension

Default File Extension Hijacking???

```\SOFTWARE\MICROSOFT\WINDOWS\CURRENTVERSION\EXPLORER\FILEEXTS```

seems malicious?

```HKLM\Software\Policies\Microsoft\Internet Explorer\Main\FeatureControl\FEATURE_HTTP_USERNAME_PASSWORD_DISABLE```


```MACHINE\SOFTWARE\WOW6432NODE\MICROSOFT\INTERNET EXPLORER\MAIN\FEATURECONTROL\FEATURE_LOCALMACHINE_LOCKDOWN```

#### dlls

```vol -f test.raw --profile=Win7SP1x64 dlllist -p 2344 ```

shell32.dll: gui stuff

propsys.dll : related to search indexing in windows

setupapi.dll: a library which installers and setup applications use. so this process is probably a dropper?


0x00000000752e0000             0xc000             0xffff 2021-03-31 22:06:22 UTC+0000   C:\Windows\syswow64\CRYPTBASE.dll

0x0000000074750000            0x16000                0x1 2021-03-31 22:07:49 UTC+0000   C:\Windows\system32\CRYPTSP.dll

0x00000000771e0000           0x11d000                0x1 2021-03-31 22:08:50 UTC+0000   C:\Windows\syswow64\CRYPT32.dll

it's using cryptographic libraries so it's probably the process responsible for encrypting?

## Process 2332 icacls.exe

```icacls.exe``` : windows exe capable of displaying and modifying the security descriptors on folders and file

the sample uses it as follows:

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/ryuk/4.png)

here are the description of the flags used: ``` /T /C /Q```

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/ryuk/5.png)

the ```F``` flag

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/ryuk/6.png)

so it's basically granting full access to all users on all files/folders in ```C:```

## Process 2424 and 1380 and 2488
 has no interesting files/keys/mutants

## getlastmodkey output

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/ryuk/7.png)


### netscan

?????

```vol -f test.raw --profile=Win7SP1x64 netscan```

0x8d1d97f0         UDPv4    0.0.0.0:60162                  *:*                                   1984     clJeKATValan.e 2021-03-31 23:53:12 UTC+0000











