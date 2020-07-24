---
title: "MyDoom.A Analysis"
layout: post
date: 2020-07-24 10:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Reverse
- Malware
- Analysis
category: blog
author: coreflood
description: ~.~
---

# Unpacking
On running ```file``` on the sample we can see it's UPX packed

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/2020-07-18-my-doom-analysis/1.png)

so we use ```upx -d sample.exe``` to unpack it.

# Imports
Seeing the list of imports it probably edits and creates a registry key using ```RegSetValueExA``` and ```RegCreateKeyExA```

there are also network connections indicators:

![](https://raw.githubusercontent.com/AlyaGomaa/Malware-Analysis/master/MyDoom.A/images/3.png)

it also create a mutex using ```CreateMutexA``` and spawns another process using ```CreateProcessA```
and searchs through the filesystem using ```FindNextFileA``` and ```FindFirstFileA```

# String Encryption

The sample uses the function at ```0x4A465E```, it takes an address and an encrypted string. looking at the cross references to this function implies that it's definitely a string decryptor as it gets called from too many placess across the binary

![](https://raw.githubusercontent.com/AlyaGomaa/Malware-Analysis/master/MyDoom.A/images/decoder.png)

the decryption algorithm is pretty simple it's actually rot13

![](https://raw.githubusercontent.com/AlyaGomaa/Malware-Analysis/master/MyDoom.A/images/rot13.png)

```python
import codecs
def decryptor(s):
    return codecs.decode(s,'rot13')
```

so i decrypted the strings and added them as comments using IDAPython to speed up the reversing process

```python 
from idc import *
from idautils import * 
import codecs

def get_function_arg(addr):
    while True:
        addr = PrevHead(addr) #addr of the first push
        addr= prev_head(addr) #addr of the second push
        if GetMnem(addr) == "push":
            break
    offset = GetOperandValue(addr, 0) 
    
    if offset=='esi' :
        addr = prev_head(addr) #get the push right before strlen function call
        offset = GetOperandValue(addr, 0)
    
    if is_loaded(offset): # offset is a valid offset
        size = get_item_size(offset)
        encrypted_str = get_bytes(offset, size)
    else: encrypted_str = offset
         
    return encrypted_str # string @ offset

decryption_function= 0x4A465E
xrefs = CodeRefsTo(decryption_function, 0)

for fun_call in xrefs:
    enc_str = get_function_arg(fun_call)
    dec_str =  codecs.encode(enc_str,'rot13') if type(enc_str)==str else enc_str
    set_cmt(fun_call , str(dec_str), 0)    
```
# Execution

# Check First Run
the sample attemps to open ```Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\Version``` subkey of the root key HKEY_LOCAL_MACHINE _(0x80000002)_ once and HKEY_CURRENT_USER _(0x80000002-0x1)_ once and creates them if it failed to open them in attempt to find out if this is the first time it executes or not.

![](https://raw.githubusercontent.com/AlyaGomaa/Malware-Analysis/master/MyDoom.A/images/open-regkey.png)

if it managed to open any of them it knows it's already been executed before so it creates the mutex ```SwebSipcSmtxS0``` to make sure only one instance of it is running.

# First Execution
the sample creates and opens a text file called ```Message``` full of garbage at the Temp Directory ```C:\Users\<username>\AppData\Local``` 

![](https://raw.githubusercontent.com/AlyaGomaa/Malware-Analysis/master/MyDoom.A/images/message.png)


# Dropped Files
The sample drops a dll called ```shimgapi.dll``` in the system directory which serves as a backdoor.

![](https://raw.githubusercontent.com/AlyaGomaa/Malware-Analysis/master/MyDoom.A/images/dropped-dll.png)


It copies itself in the system directory under the name of ```taskmon.exe``` which was a valid windows process back then.

![](https://raw.githubusercontent.com/AlyaGomaa/Malware-Analysis/master/MyDoom.A/images/taskmon.png)


# Persistence 
It creates the registry key ```TaskMon``` in ```Software\Microsoft\Windows\CurrentVersion\Run``` to automatically run during system startup without requiring user interaction.

![](https://raw.githubusercontent.com/AlyaGomaa/Malware-Analysis/master/MyDoom.A/images/persistence.png)


# Start Execution
The sample gets the current system time and date to make sure the day is after ```2004-02-01``` and the time is after  ```16:09:18``` before it starts execution. hence not every infected machine will perform the attack, it depends on the system time.

![](https://raw.githubusercontent.com/AlyaGomaa/Malware-Analysis/master/MyDoom.A/images/start-date.png
)

to get the hardcoded time i converted the two 32-bit values at ```dwLowDateTime``` and ```dwHighDateTime``` to a 64-bit value _(filetime format)_ 


```python
dwLowDateTime = 0x0BE9ECB00
dwHighDateTime = 0x1C3E8DD << 32
execution_filetime = (dwLowDateTime & 0xFFFFFFFF) + dwHighDateTime
```
then i used [this](https://gist.github.com/Mostafa-Hamdy-Elgiar/9714475f1b3bc224ea063af81566d873
) script to convert it to datetime.


# DoS Attack
First it will check if the machine is connected to the internet, then it creates 63 threads and sends a 'GET /HTTP/1.1' request from each one of them to ```www.sco.com``` on TCP port 80

![](https://raw.githubusercontent.com/AlyaGomaa/Malware-Analysis/master/MyDoom.A/images/packetss.png)


# Spreading (Kazaa Spread)
> Kazaa is a peer-to-peer file sharing application program for the Microsoft
Windows platform

The worm copies itself under one of the following names along with a random extension in the user shared folder on the kazaa network.

## Names:
- winamp5
- icq2004-final
- activation_crack
- strip-girl-2.0bdcom_patches
- rootkitXP
- office_crack

## Extensions: 
.exe , .src , .pif , .bat

# Mass Mailing 

## Generated Email example
![](https://raw.githubusercontent.com/AlyaGomaa/Malware-Analysis/master/MyDoom.A/images/email.png)

### Generating Sender Email
it adds one of the following names:
"john" ,"john" ,"alex" ,"michael" ,"james" ,"mike" ,"kevin" ,"david" ,"george" ,"sam" ,"andrew" ,"jose" ,"leo" ,"maria" ,"jim" ,"brian" ,"serg" ,"mary" ,"ray" ,"tom" ,"peter" ,"robert" ,"bob" ,"jane" ,"joe" ,"dan" ,"dave" ,"matt" ,"steve" ,"smith" ,"stan" ,"bill" ,"bob" ,"jack" ,"fred" ,"ted" ,"adam" ,"brent" ,"alice" ,"anna" ,"brenda" ,"claudia" ,"debby" ,"helen" ,"jerry" ,"jimmy" ,"julie" ,"linda" ,"sandra"
or any 1-3 random characters

to one one the following domains
- "aol.com"
- "msn.com"
- "yahoo.com"
- "hotmail.com" 

### Generating Attachment
it makes a copy of itself under one of the followng names:
document, readme, doc, text, file, data, test, text, message, body

and adds one or two of the following extensions: 
pif, scr, exe, cmd, bat, HTM, TXT or DOC 

### Generating Message

It chooses one of the following subjects or any 3 to 16 random letters
- test
- hi
- hello
- Mail Delivery System
- Mail Transaction Failed
- Server Report
- Status
- Error

and one of the following messages:
- "test"
- "The message cannot be represented in 7-bit ASCII encoding and has been sent as a binary attachment."
- "The message contains Unicode characters and has been sent as a binary attachment."
- "Mail transaction failed. Partial message is available."

# Gathering E-mail Addresses
The worm looks in ```HKCU\Software\Microsoft\WAB\WAB4\Wab File Name``` _(Windows Address Book)_ which contains emails saved on the machine and skips emails that contain: .edu 

It then scans internet temp directories ```C:\Windows\Temporary Internet Files``` and ```C:\Users\<username>\AppData\Local\Internet Temporary Files``` for valid emails 

and then it scans all drives for files with the following extensions:
txt ,htm ,sht ,php ,asp ,dbx ,tbb ,adb, pl , wab

It skips emails with these domains:
"avp", "syma", "icrosof", "msn.", "hotmail", "panda","sopho", "borlan", "inpris", "example", "mydomai", "nodomai","ruslis", ".gov", "gov.", ".mil", "foo."


checks that the sender and the reciever's emails don't contain any of the following substrings:
anyone, bugs, ca, contact, feste, gold-certs, help, info, me, no, nobody, noone, not, nothing, page, postmaster, privacy, rating, root, samples, service, site, soft, somebody, someone, submit, the.bat, webmaster, you, yours , "berkeley", "unix", "math", "bsd", "mit.e", "gnu", "fsf.", "ibm.com", "google", "kernel", "linux", "fido", "usenet","iana", "ietf", "rfc-ed", "sendmail", "arin.", "ripe.", "isi.e", "isc.o", "secur", "acketst", "pgp", "tanford.e", "utgers.ed", "mozilla" , "admin", "icrosoft", "support", "ntivi", "unix", "bsd", "linux", "listserv", "certific", "google", "accoun", abuse , secur ,spam
 , www 

# Expiry Date

The sample gets the current system time and compares it to ```2004-02-12```  at ```a1+0x224``` to make sure it doesn't execute the malicious code after that date. However, the backdoor ```shimgapi.dll``` remains open.

![](https://raw.githubusercontent.com/AlyaGomaa/Malware-Analysis/master/MyDoom.A/images/kill-switch.png)

# Shimgapi.dll

## Unpacking
It's also UPX packed so we use ```upx -d shimgapi.dll ``` to unpack it

## Persistence
First it calls ```RegisterServiceProcess``` which was a valid windows api in the Windows 9x based OS used to register a process as a service in order to hide itsef from the Task Manager.

then it adds itself to the registry key ```CLSID\\{E6FB5E20-DE35-11CF-9C87-00AA005127ED}\\InprocServer32```. by creating this entry windows will load it in the address space of explorer.exe

## Backdoor (Keeping Access)
It then tries to open any port in the range of 3127 to 3198, this open port acts as the backdoor so that any user can exploit it later.

![](https://raw.githubusercontent.com/AlyaGomaa/Malware-Analysis/master/MyDoom.A/images/open-port.png
)

before running the dll:

![](https://raw.githubusercontent.com/AlyaGomaa/Malware-Analysis/master/MyDoom.A/images/netstat-before.png)

after running the dll:

![](https://raw.githubusercontent.com/AlyaGomaa/Malware-Analysis/master/MyDoom.A/images/netstat-after.png)


# References
 - [https://github.com/yorickdewid/MyDoom](https://github.com/yorickdewid/MyDoom)
 - [https://www.giac.org/paper/gcih/568/mydoom-dom-anlysis-mydoom-virus/106069](https://www.giac.org/paper/gcih/568/mydoom-dom-anlysis-mydoom-virus/106069)
 - [https://github.com/inforion/idapython-cheatsheet](https://github.com/inforion/idapython-cheatsheet)


# Download the sample
you can [download the sample here](https://malshare.com/sample.php?action=detail&hash=53df39092394741514bc050f3d6a06a9).
