---
author: virtualfrog
comments: true
date: 2017-07-31 09:23:35+00:00
layout: post
link: https://virtualfrog.wordpress.com/2017/07/31/vmware-releases-6-5-update-1/
slug: vmware-releases-6-5-update-1
title: VMware releases 6.5 Update 1
wordpress_id: 598
categories:
- General
- VMware
---

A couple of days ago VMware released version 6.5 Update 1 for some of their products:



	
  * vCenter Server: [Link](https://my.vmware.com/group/vmware/details?downloadGroup=VC65U1&productId=614&rPId=17343)

	
  * ESXi: [Link](https://my.vmware.com/group/vmware/details?downloadGroup=ESXI65U1&productId=614&rPId=17343), [Link to HPE Custom Image](https://my.vmware.com/group/vmware/details?downloadGroup=OEM-ESXI65U1-HPE&productId=614)

	
  * Site Recovery Manager (SRM): [Link](https://my.vmware.com/group/vmware/details?downloadGroup=SRM651&productId=617&rPId=17333)


I was waiting for Update 1 of the vSphere Stack for a while now, and the List of bugfixes has quite a few entries with critical bug fixes.

<!-- more -->


## Installation Update 1 on 6.5 VCSA


The Update process is incredibly fast. Attach the .iso to your VCSA VM. Go the VAMI (https://<your-vcenter>:5480) and select the Update Tab. Choose to check for Updates on CD-ROM and then install the updates.

After about 4 minutes the installation is done and you'll need to manually reboot the appliance. That's it you're done.

Upgrading from vSphere 6.0 Update 3 is now fully supported with 6.5 Update 1.


## Upgrading ESXi hosts to 6.5 Update 1


With ESXi hosts there are many roads to choose from.



	
  * Patch using VMware Update Manager

	
  * Upgrade using VMware Update Manager (Upload the iso file)

	
  * Re-Install using the latest image and kickstart script


In my Lab where we still use HPE G7 hardware (I know, it's not supported, but it still works and it's a lab..) I had a couple of problems. Using the iso with VUM did not work. Next I got a purple screen when doing a fresh install with the latest HPE Image (see link above). The exception was of type 14 (seems to be a page fault according to VMware KBs) so I'm not sure if it's because the hardware is not supported anymore or if I have some hardware issues.

Anyways, I'll be working on that and I'm sure I'll get it to work and update my findings here.


## Upgrading SRM to 6.5 Update 1


For this upgrade to work you'll need to be on the 6.1.2 Build at least. Upgrading from a 6.0 Installation is not supported because you will run into problems. See the [Release Notes](https://t.co/3NRHCLZbOS) for further details.
