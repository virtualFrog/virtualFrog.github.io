---
author: virtualfrog
comments: true
date: 2019-02-13 07:35:57+00:00
excerpt: A short story about a support request that involved VMware workstation, a
  Windows XP VM and a custom keyboard for a perfectly good use case.
layout: post
link: https://virtualfrog.wordpress.com/2019/02/13/how-to-make-sure-your-usb-device-always-gets-passed-through-to-your-vm/
slug: how-to-make-sure-your-usb-device-always-gets-passed-through-to-your-vm
title: How To make sure your USB Device always gets passed through to your VM
wordpress_id: 1191
categories:
- General
- vExpert
- VMware
---

Last week we received a support request from one of our customers. Their problem was that in their Windows XP (!) machine a special keyboard was not always recognised and we were asked to help them fix that.<!-- more -->

Now, before I receive comments about how outdated and unsupported Windows XP is these days let me explain their perfectly valid use case:


## Use Case for Windows XP in 2019


The company makes huge industrial machines which need some sort of software to work properly. These huge machines have a very long lifespan and sometimes the software written for these machines is back from the windows 95 era. Now, the customer needs to support these old machines as well as his latest models. So the customer was building a new “industrial PC” which is basically a hardened housing for a small computer where this software needs to run. The problem here was that you would only get windows 10 installed on the new computers and the old software was impossible to install on windows 10. The “latest” windows that the software is able to run on is in fact windows XP. That would explain their use case to use Windows XP today.

The customer has already made a great effort and had a lot of things automated. As soon as the computer booted up it would start the Windows XP VM in a VMware workstation KVM instance (KVM instance hides the bar on top and actually kind of hides the fact that you are running in a VM). And when the customer shut down the VM the underlying host (windows 10) would detect the shutdown of the VM and initiate a shutdown of itself as well. Quite brillant I must say.


## The Problem: USB Keyboard not always connecting


The problem the customer was facing now was that a special kind of keyboard would sometime be passed through to the VM and sometimes not. When it would not connect all the old software would error out and not start at all. Now imagine an industry working man who only wants to use this industrial machine and has not much knowledge in “computers and such” and how such errors would freak them out especially when they would occur frequently.

Now at this point I have to admit I had know idea what I was doing when I was called into action on this support request. My first thought was that maybe windows XP is at fault and if so no one will be able to help the customer out. But anyways, I went on-site and looked at the situation. Most of the information above I was only able to get when I was on-site and talking to the customer directly.

We did some testing and found out that in the case where the VM did not have the keyboard passed through the underlying windows 10 was perfectly able to work with the keyboard. So it became clear pretty quickly that we need to configure this special USB device to always pass through to the VM. How do I do that? I remembered on my VMware Fusion that it always asks me what I want to do with a USB device when I plug it in while running a VM. This was somehow not the case here. I figured I would take a look at the .vmx file of the windows XP VM and see how it was “configured” there. The “Settings” of the VM did not help at all because I could only configure a USB Controller for the VM and nothing else.

In the vmx file there was a line that looked like this:


<blockquote>usb.autoConnect.device0 = “path:2/1/1/1 autoclean:1"</blockquote>


So the USB device should be passed through to the VM if the USB device was found at the given path. That was not an optimal solution and would assume/require that the USB device was always found at the same path. So after a little digging I found a KB article from VMware regarding this functionality: [https://kb.vmware.com/s/article/1648](https://kb.vmware.com/s/article/1648)

It was actually a very good article that discussed how to automatically connect USB devices to a VM. There are a lot of options to achieve this:



	
  * path

	
  * Vendor ID and/or Product ID

	
  * Name


And you are even free to mix all of the above into on string.

I talked with the customer and he said that they had different keyboards in use so a PID (Product ID) would probably not apply to all of them. For the same reason "Name" was ruled out (and we later found out that windows 10 did not recognise the name of the device anyways) and "Path" was a bad idea to begin with but was actually the way that workstation automatically added the line in the .vmx file.

Luckily you can limit yourself to just the Vendor ID (VID). And we decided this would probably work best.


## Finding VID and PID of a USB Device


How do I find out the Vendor ID of my device? The [KB article](https://kb.vmware.com/s/article/1648) has some good details about that as well, especially on a linux host this would be easy. For windows it describes to go looking in the registry and search for the name of the device in a given location of the registry. Here is where we found out that windows 10 does not list or recognise the name of the device. Looking through the registry was therefore not very ideal.

In comes the device manager. The good old view accessed with a right click on the "start" button or by directly running "devmgmt.msc":

![Device-Manager](https://virtualfrog.files.wordpress.com/2019/02/device-manager.png)

This will open this view:

![Device-Manager-Open](https://virtualfrog.files.wordpress.com/2019/02/device-manager-open.png)

Now in order to find out the VID (and or PID) of a device you just need to find it, right click it and go into its properties:

![Screenshot 2019-02-13 at 08.17.14](https://virtualfrog.files.wordpress.com/2019/02/screenshot-2019-02-13-at-08.17.14.png)

Once the "Properties" have been opened you just switch to the "Details" Tab and change the propery to "Hardware Ids":

![Screenshot 2019-02-13 at 08.17.52](https://virtualfrog.files.wordpress.com/2019/02/screenshot-2019-02-13-at-08.17.52.png)

Here you can see that VID and PID are displayed in the string shown in the value field.

In this example the Vendor ID (VID) is: 0E0F and the Product ID (PID) is: 0003.

This would translate to a vmx file configuration like this:


<blockquote>usb.autoConnect.device0 = “vid:0E0F pid:0003 autoclean:1"</blockquote>


If you only wanted to use the Vendor ID like we decided to in our case:


<blockquote>usb.autoConnect.device0 = “vid:0E0F autoclean:0"</blockquote>





## Autoclean Parameter


You might ask yourself: What is autoclean?

Good question, let me quote the [KB article](https://kb.vmware.com/s/article/1648) directly:


<blockquote>

> 
> autoclean:1 - autoconnect if a device matches the pattern, removed if the VM is powered on and no device matches the pattern, removed if disconnected through the UI.
> 
> </blockquote>







<blockquote>autoclean:0, no autoclean - autoconnect if a device matches the pattern, not removed if the VM is powered on and no device matches the pattern, not removed if disconnected through UI. Although the autoconnect entries are not removed, the device will not autoconnect again after the user disconnects the device using the UI for that session of the VM. The same device will autoconnect when the VM is restarted or when the device is physically unplugged and replugged into the same port on the host.</blockquote>


We tested and decided that for this industrial pc use case the "autoclean:0" option is a better fit.


## The solution


So in the end our string looked like this:


<blockquote>usb.autoConnect.device0 = “vid:0E0F autoclean:0"</blockquote>


And we successfully verified the behavior by countless reboots and connected the keyboard on different USB ports.

I have learned a lot on this support request. I do not often see workstation use cases but as a VMware expert  in my company I get asked when there is a problem and VMware even just mentioned somewhere. I am glad I took this request, even gladder I was able to find a solution and make our customer happy. I hope this post will help someone down the road with a similar use case.
