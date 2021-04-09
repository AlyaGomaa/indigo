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

1.png

it migh be injecting into the calculator but we'll see..

### Privileges
``` vol -f test.raw --profile=Win7SP1x64 privs -p 2344,2332,2488,2424,1380, | grep Enabled ``` to check enabled priveledges

2.png

nothing interesting here since the only priveledge is enabled by default for all of them 

### Process 2344 xxx.exe

#### files
``` vol -f test.raw --profile=Win7SP1x64 handles -p 2344 -t file```
nothing interesting was found here

#### Mutants
```vol -f test.raw --profile=Win7SP1x64 handles -p 2344 -t mutant```
shows that it creates 5 mutexes

3.png

####  keys
```vol -f test.raw --profile=Win7SP1x64 handles -p 2344 -t key```

show that it access the network provider using the key 
```MACHINE\SYSTEM\CONTROLSET001\CONTROL\NETWORKPROVIDER\HWORDER```

this is where the user specifies the default program to open files with a specific extension
```\SOFTWARE\MICROSOFT\WINDOWS\CURRENTVERSION\EXPLORER\FILEEXTS```

seems malicious?
```HKLM\Software\Policies\Microsoft\Internet Explorer\Main\FeatureControl\FEATURE_HTTP_USERNAME_PASSWORD_DISABLE```



```MACHINE\SOFTWARE\WOW6432NODE\MICROSOFT\INTERNET EXPLORER\MAIN\FEATURECONTROL\FEATURE_LOCALMACHINE_LOCKDOWN```

shell32.dll: gui stuff
propsys.dll : related to search indexing in windows
setupapi.dll: a library which installers and setup applications use. so this process is probably a dropper?



```dlllist```
0x0000000074750000            0x16000                0x1 2021-03-31 22:07:49 UTC+0000   C:\Windows\system32\CRYPTSP.dll
0x00000000771e0000           0x11d000                0x1 2021-03-31 22:08:50 UTC+0000   C:\Windows\syswow64\CRYPT32.dll



## Process 2332 icacls.exe

capable of displaying and modifying the security descriptors on folders and file
the sample uses it as follow:
4.png

5.png
6.png


## Process 2424 and 1380 and 2488
 hase no interesting files/keys/mutants















