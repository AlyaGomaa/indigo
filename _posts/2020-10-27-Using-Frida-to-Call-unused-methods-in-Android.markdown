---
title: "Using Frida to Call unused methods in Android"
layout: post
date: 2020-10-27 07:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Reverse
category: blog
author: coreflood
description: ----
---

Frida is a dynamic instrumentation toolkit that allows us to run javascript code inside android apks.


# Installing Frida

### Genymotion
We're going to be using [Genymotion](https://www.genymotion.com/download/) as our emulator so we don't have to root a physical device. just download any type of device with Android 6.0.


#### Frida
``` pip3 install frida-tools ``` and ``` pip3 install frida``` to install frida.

you'll need a suitable version of frida's android server for this tutorial i'll be using ```frida-server-14.0.1-android-x86``` you can download it here[https://github.com/frida/frida/releases]

#### adb
you can download the latest version of adb [here](https://www.xda-developers.com/install-adb-windows-macos-linux/).

--- 

First you need to **connect adb to your emulator** , the device IP is written in the title bar

so ```adb connect 192.168.56.101:5555``` to connect to it. make sure the device is running or adb will fail to connect.

**to run frida-sever:** 

``` ./adb push frida-server /data/local/tmp/ ``` to push the server to your android device

``` ./adb shell ```  to start an adb shell

``` chmod 755 /data/local/tmp/frida-server ```
don't use chmod +x because some versions of chmod don't understand the o+rw notation.

```cd /data/local/tmp ``` 

```./frida-server & ``` run the server in the background

---

We'll be solving [this](https://github.com/AlyaGomaa/blog/blob/gh-pages/_posts/frida-challenge/is_it_really_random.apk) Challenge to demonstrate how to use frida to call a method that is not called anywhere in an android apk.

we first need to install the app in genymotion, you can drag and drop it or use ``` adb push is_it_really_random.apk /data/local/tmp  ``` .

I'll be using jadx-gui to view the source code.

If we take a look at the ```MainActivity``` class we'll see a method called ```decryptIfYouCan```, looking at the cross references we see that it's not called anywhere in the code so hooking it would be useless. it only calls ```finsh_the_work_plz``` and passes it a random number

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/frida-challenge/usage.png)

```finsh_the_work_plz``` uses the generated random number to get the encrypted data from a random cell in a database that has 20000 records in the apk and sends it along with the key and the IV to ```decrypt1``` and ```decrypt2``` to decrypt the flag.

```
  private String finsh_the_work_plz(int random_posstion) {
        String sel = random_posstion + "";
        String h1 = this.db.Getdata(sel, "half");
        String IV1 = this.db.Getdata(sel, "IV");
        String key = this.db.Getdata(sel, "key");
        return this.de.decrypt2(this.de.hexStringToByteArray(this.de.decrypt1(this.db.Getdata(sel, "encrypted"), this.db.Getdata(h1, "IV"))), this.de.hexStringToByteArray(IV1), this.de.hexStringToByteArray(key));
    }
```
since the challenge generates a random number in the range 0 to 15000 we can safely assume that the flag will be in the last 5000 cells (to speed up the bruteforce)

to use frida we need to know the package name, it's listed in ```AndroidMainfest.xml``` 

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/frida-challenge/mainfest.png)

To call a method that hasn't been called we need to know its class name (in this case ```MainActivity```),find an instance created by that class using frida's ``` Java.choose( packageName.className , callback functions) ```, and call the method on that instance.

annnnd to solve the challenge we simply need to bruteforce the random number.

here's the Javascript code we'll be injecting into the app

```
Java.perform(function x(){

Java.choose("com.example.random_random_randomly.MainActivity" , { //class name

	onMatch : function(instance){
	 // This function will be called for every instance found by frida
  console.log("Instance: " + instance );
	var i = 15001
	while(1){// Bruteforce the random number

		console.log(i)
		try{
			// call the method on the found instance
			var returnValue = instance.finsh_the_work_plz(i) 
			if(returnValue.includes("not flag")){
				console.log("Wrong number: " + i);
			} else {
				console.log("Correct: " + i )
				console.log(returnValue)
				break;
			}
		} catch(err){
			continue;
		}
		i+=1;
		
	}
  },
  onComplete : function(){} //will be called when the method is completed.
});
})
```

make sure the frida-server is running and then ``` frida -U com.example.random_random_randomly -l frida123.js ``` in a new terminal to run the script.

![](https://raw.githubusercontent.com/AlyaGomaa/blog/gh-pages/_posts/frida-challenge/flaag.png)

everything inside ```Java.perform``` will be injected into our app.

i wrapped my code inside a giant try and catch because there are some integers that frida simply doesnt like and it crashes? probably a bug in frida


> Data types are divided into two groups:
> - Primitive data types - includes byte, short, int, long, float, double, boolean and char
> - Non-primitive data types - such as String, Arrays and Classes 


```finsh_the_work_plz``` takes an int (primitive) that's why we didn't need to create an instance of it.

if we wanted to call ```decryptIfYouCan(View view)``` for example we would need to create an instance of the View class and then pass that instance to to ```decryptIfYouCan```

```
var view_class = Java.use("android.view.View"); //hook the View class
var view_instance = view_class.$new(); // create an instance

```
```
var returnValue = instance.finsh_the_work_plz(view_instance);
```

# References

[https://11x256.github.io/](https://11x256.github.io/)

:)

