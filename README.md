# ingenic-t31-sdk-camera
Using the Ingenic T31 SDK to access the camera sensor via SDK, buildroot and openipc  

<!--
I am a jack-of-all-trades engineer with more than 30 years of experience. Most of this is in hardware and firmware design 
of microcontroller based devices. Extensive background in Python, c (not recent) and Matlab. More recently, computer vision 
and limited embedded Linux. I use chat-gpt4 extensively. I find it extremely helpful for new areas I have limited experience 
in (eg. Linux, c++) but less useful as my knowledge increases in a given field.  
-->

## Links  
Ingenic T31 SDK_20221229  
https://github.com/OpenIPC/firmware for openIPC Apr 18, 2021  
https://github.com/gtxaspec/ingenic-vidcap for code that does part of what I want to do  
https://buildroot.org/downloads/buildroot-2023.11.1.tar.gz openipc is built on top of buildroot  

## Discussion
Eventually I want to write code that accesses the camera, gets frames and stores in a FIFO buffer.  
I want to use gdbserver for remote debug and opencv for image processing. There will be no AI computer vision as such on the device.  

The SDK clearly shows the kernel used as version 3.10.14 in its Makefile.

``` makefile
VERSION = 3
PATCHLEVEL = 10
SUBLEVEL = 14
EXTRAVERSION =
NAME = TOSSUG Baby Fish
```

There are two versions of compilers included in the SDK (4.7.2 and 5.4.0).  
When configuring buildroot to use a provided compiler, the make fails with the following errors:  

Incorrect selection of kernel headers: expected 3.10.x, got 3.5.x  
Incorrect selection of kernel headers: expected 3.10.x, got 4.2.x  

for gcc 4.7.2 and 5.4.0 respectively.  
A sample config for openipc is provided (t31_tsardoz_defconfig).  
There may be errors in this config file causing my problems.  

Using chat-gpt4, it suggested I look at the version.h files provided with these compilers.  
I did so and found the following.  

./gcc_472/mips-gcc472-glibc216-64bit/mips-linux-gnu/libc/uclibc/usr/include/linux/version.h  
```.h
#define LINUX_VERSION_CODE 197892
#define KERNEL_VERSION(a,b,c) (((a) << 16) + ((b) << 8) + (c))
```
and  

./gcc_540/mips-gcc540-glibc222-64bit-r3.3.0/mips-linux-gnu/libc/uclibc/usr/include/linux/version.h

```.h
#define LINUX_VERSION_CODE 262656
#define KERNEL_VERSION(a,b,c) (((a) << 16) + ((b) << 8) + (c))
```

It turns out that these codes correspond to KERNEL_VERSION 3.5.4 and 4.2.0 respectively.  
I interpret this as meaning the included precompiled libraries have been compiled with these kernels, not 3.10.14.  
I could be wrong but this seems logical.  
These are the same as the error messages I got from the openipc make earlier.
Whilst GPT4 suggested I look at these version.h files, it has nothing to do whatsover about interpretation of their contents.  

My goal is to write c/c++ code to use these libraries to access the camera.  
It seems to me that I cannot get buildroot to access these provided compilers as their kernel headers are inconsistent with the actual kernel 3.10.14 and throws the above errors. 
So unless I have made a mistake in my openipc config file (entirely possible), I think my best approach might be to use the compiler with headers closest 
to 3.10.14, which is 4.7.2 (headers 3.5.4). I can tell openipc to make its own compiler and not use an external one, using uclibc. I presume I could not actually use this compiler built by buildroot/openipc to access the libraries given in the SDK (maybe I can?). 

Can I just tell buildroot/openipc to use kernel headers 4.2.0 or 3.5.4 and forget 3.10.14? Then make one of these kernels and not 3.10.14?  

gtxaspec used static libraries and chatgpt4 suggested I do the same thing for best compatibility.  

Comments welcome on how to best approach this. Maybe my config file can be fixed, or maybe there is an alternative approach I have not considered. Or I am completely wrong about everything ...  

