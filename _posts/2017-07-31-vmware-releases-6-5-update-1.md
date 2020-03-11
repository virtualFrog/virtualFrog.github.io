---
layout: post
title: VMware releases 6.5 Update 1
date: 2017-07-31 11:23:35.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- virtualFrog Posts
- VMware
tags: []
meta:
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:56:"https://twitter.com/virtual_dd/status/891952478232743936";}}
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _thumbnail_id: '1352'
  _publicize_job_id: '7705723725'
  _publicize_done_9340485: '1'
  _wpas_done_9383931: '1'
  publicize_twitter_user: virtual_dd
  publicize_linkedin_url: https://www.linkedin.com/updates?discuss=&scope=391645417&stype=M&topic=6297718171214323712&type=U&a=hfV6
  _publicize_done_14862392: '1'
  _wpas_done_14749148: '1'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2017/07/31/vmware-releases-6-5-update-1/"
---
A couple of days ago VMware released version 6.5 Update 1 for some of their products:

- vCenter Server: [Link](https://my.vmware.com/group/vmware/details?downloadGroup=VC65U1&productId=614&rPId=17343)
- ESXi: [Link](https://my.vmware.com/group/vmware/details?downloadGroup=ESXI65U1&productId=614&rPId=17343), [Link to HPE Custom Image](https://my.vmware.com/group/vmware/details?downloadGroup=OEM-ESXI65U1-HPE&productId=614)
- Site Recovery Manager (SRM): [Link](https://my.vmware.com/group/vmware/details?downloadGroup=SRM651&productId=617&rPId=17333)

I was waiting for Update 1 of the vSphere Stack for a while now, and the List of bugfixes has quite a few entries with critical bug fixes.

<!--more-->

## Installation Update 1 on 6.5 VCSA

The Update process is incredibly fast. Attach the .iso to your VCSA VM. Go the VAMI (https://\<your-vcenter\>:5480) and select the Update Tab. Choose to check for Updates on CD-ROM and then install the updates.

After about 4 minutes the installation is done and you'll need to manually reboot the appliance. That's it you're done.

Upgrading from vSphere 6.0 Update 3 is now fully supported with 6.5 Update 1.

## Upgrading ESXi hosts to 6.5 Update 1

With ESXi hosts there are many roads to choose from.

- Patch using VMware Update Manager
- Upgrade using VMware Update Manager (Upload the iso file)
- Re-Install using the latest image and kickstart script

In my Lab where we still use HPE G7 hardware (I know, it's not supported, but it still works and it's a lab..) I had a couple of problems. Using the iso with VUM did not work. Next I got a purple screen when doing a fresh install with the latest HPE Image (see link above). The exception was of type 14 (seems to be a page fault according to VMware KBs) so I'm not sure if it's because the hardware is not supported anymore or if I have some hardware issues.

Anyways, I'll be working on that and I'm sure I'll get it to work and update my findings here.

## Upgrading SRM to 6.5 Update 1

For this upgrade to work you'll need to be on the 6.1.2 Build at least. Upgrading from a 6.0 Installation is not supported because you will run into problems. See the [Release Notes](https://t.co/3NRHCLZbOS) for further details.

