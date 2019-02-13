---
author: virtualfrog
comments: true
date: 2017-05-02 13:04:47+00:00
layout: post
link: https://virtualfrog.wordpress.com/2017/05/02/introducing-the-new-esxi-staging-appliance/
slug: introducing-the-new-esxi-staging-appliance
title: Introducing the new ESXi Staging Appliance
wordpress_id: 105
categories:
- General
- Kickstart
- Scripting
---

A couple of years ago I made a post about the "REDA - Rapid ESXi Deployment Appliance" which was basically a customized version of the old "EDA" which was available to download from the VMware solutions exchange.

When face with a environment that deal with multiple clusters and lots of host the old EDA/REDA was very limited in terms of scalability. You could only have one script, one ISO image and had to put all of yours hosts into one big list.<!-- more -->

Two years ago I switched jobs from internal IT to a service provider specialising in VMware datacenter design. So naturally I had to install lots and lots of ESXi hosts and was very frustrated with the scalability issues of the EDA/REDA.

So I went to work on a new appliance. The new one is called "xSA - Esxi Staging Appliace" and has a ton of new features.



	
  * You can add as many environments (aka clusters) as you want

	
  * each environment can have its own iso image and kickstart script

	
  * you can choose to either install over intergrated pxeboot, custom ISO image or usb stick

	
  * the GUI features a very nice script editor

	
  * backup and restore of the appliance is included

	
  * everything resides inside a database


that's just describing version 1.3.1 which we currently ship to our customers.

the heart of the appliance is the kickstart script. the one we're working on has grown to 1000 lines of code and can basically do everything you can configure on a esxi host manually (no part of vCenter).

the latest function I added was to make the esxi host join a vcenter upon installation (the script to do this wasn't mine, I found it on William Lam's Github).

Unfortunately I developed the appliance on my company's time so I'm not allowed to share it outside of my company. But I'd be available to give you tipps and such if you're looking into the same kind of appliance technology.

What I am allowed to share is a video of the appliance in action:
https://www.youtube.com/watch?v=SMUITgmjQWo

Update:

Since this post is getting a lot of views I thought I'd share some more information about the xSA. Here is the roadmap for features to come. We're currently at version 1.4 and I'm planning to work on 2.0 for release at the end of the year.

2.0:

	
  * Integrate HPE SPP Images to make available over pxeBoot

	
  * Have a powershell script that reads out all the information on a existing infrastructure and feed it into the database of the xSA

	
  * Make the information exportable into a csv for reference of your host configuration

	
  * make the kickstart script easily updateable by splitting the functions and the main program part

	
  * build in ability to upgrade based on latest gitLab zip (go to gitlab, download .zip from stable branch and upload it to the xSA over the GUI

	
  * have the kickstart script configure stuff in the vCenter server over API

	
  * integrate dvSwitch configuration

	
  * clean up the GUI, resemble the Web-Client from VMware


If you have any questions, feel free to ask them here or on twitter and I will gladly give you all the information I am allowed to share.
