---
layout: post
title: Setting up a greenfield vSAN cluster on 6.5 Update 1
date: 2017-09-07 16:14:52.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- VCDX
- virtualFrog Posts
- VMware
- vSAN
tags: []
meta:
  _thumbnail_id: '437'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '9049594427'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:56:"https://twitter.com/virtual_dd/status/905796521316188160";}}
  _publicize_done_9340485: '1'
  _wpas_done_9383931: '1'
  publicize_twitter_user: virtual_dd
  publicize_linkedin_url: https://www.linkedin.com/updates?discuss=&scope=391645417&stype=M&topic=6311562214516105216&type=U&a=8Gv4
  _publicize_done_14862392: '1'
  _wpas_done_14749148: '1'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2017/09/07/setting-up-a-greenfield-vsan-cluster-on-6-5-update-1/"
---
Recently I had the pleasure of doing a vSAN project for one of our customers. The project scale was very small but nonetheless very interesting.

The project scope was:

- 4 HPE ProLiant DL 380 Gen9 Servers
  - Each had 6 SSDs (2x Cache @ 400 GB, 4x Capacity @ 1.92 TB)
- 1 vCenter Server Appliance
- Migration of VMs from existing environment (vSphere 5.5 on Gen6 hardware)

<!--more-->

The thing that was the most interesting here was that I finally got to do a greenfield vSAN environment. VMware has put a lot of thought into this "chicken and the egg" situation where you want to build a vSAN cluster from scratch and need to deploy the VCSA to start the vSAN configuration.

So, it's very straightforward. I installed the 4 hosts using my xSA (ESXi Staging Appliance) and had them up and running in less than an hour. After that I started the VCSA installer on my macbook. During the wizard you need to connect to a esxi host and then choose which storage it should be placed on. Here you can choose that you want vSAN!

After that you need to specify the datacenter and cluster name and then carry on like on any other installation. It will then first do the vSAN on one host (you'll need to specify which disks to claim for cache and which for capacity) and deploy the VCSA on top of that.

After everything is done you connect to your new vCenter Server Appliance, join the hosts to the vCenter and then configure the rest of the vSAN (like the dvSwitch with the VMkernel portgroups) and that is it.

We ran a couple of performance tests, then we did failover testing to verify the behaviour of HA, disk failures and maintenance mode. After everything succeeded we were able to move on to the migration phase of the project.

I joined the existing hosts into the new vCenter server and one after another shut down a vm and did a cold migration of both compute and storage to the new vSAN cluster. It took about 4 hours to transfer all 20 VMs.

After that the customer was happy with the increased performance and the smooth transition to the new environment. And I was happy as well because I was able to finally leverage this bootstrap method of creating vSAN on the first host and only after that stretch it to the entire cluster.

I think I'll use this project as a baseline for my VCDX project.

