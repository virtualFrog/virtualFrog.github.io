---
author: virtualfrog
comments: true
date: 2014-11-21 14:37:30+00:00
layout: post
link: https://virtualfrog.wordpress.com/2014/11/21/alternative-to-vmware-autodeploy-rapid-esxi-deployment-appliance/
slug: alternative-to-vmware-autodeploy-rapid-esxi-deployment-appliance
title: 'Alternative to VMware AutoDeploy: Rapid ESXi Deployment Appliance'
wordpress_id: 17
categories:
- General
- Kickstart
- Scripting
---

[![Flash: Extremly Fast](https://virtualfrog.files.wordpress.com/2014/11/wally76.jpg?w=99)](https://virtualfrog.files.wordpress.com/2014/11/wally76.jpg)When you're thinking about upgrading your infrastructure to the latest stable version of ESXi you have to think about how you will deploy your hosts with the new ESXi Hypervisor.

One way to it would be to use VMware Update Manager. Another way would be to use the vSphere AutoDeploy Feature combined with Host Profiles. Unfortunately not everyone can afford the required "Enterprise+" license to use this feature.<!-- more -->

Here comes the "REDA" into play. The "Rapid ESXi Deployment Appliance". This appliance was initially released by VMware labs a few years back. Actually, they released it as "EDA" ESX Deployment Appliance.  But developement halted when VMware focused on their other products like "AutoDeploy".
<!-- more -->

Our consultant has soon realised how easy it would become to install ESXi if the "REDA" was just developed a little further. I help him as best I can and provides scripts which configure parts of the hosts specific to the current client's needs. For example I wrote a script that determines the ESXi Version of the host and based on that saves the ssh-keys in the appropriate directory.

To better understand how this REDA works and what it's benefits are. Here is a short intro:

In the REDA you have a general script, which will be applied to all the hosts. Perfect place to put in functions like the one I mentioned regarding SSH-keys. Basically in this script you write functions that you want to have cluster-wide. Like if you're still using Standard vSwitches you can write functions that create those vSwiches including uplinks, portgroups and whatnot. So you can be sure that each and every host has the same PortGroup names as the rest of the cluster.

After that you configure the basic network setup (the one you would be configuring if you went through the installer) for the management network of the host. And after you've done that you can add "hosts" to your eda. Theses hosts will be selectable when you do a PXE boot on your ESXi Host. Here you will need to configure the values that are specific for each host (like IP-Addresses of the vmkernel-interfaces).

When all the configurations are done you need to setup the dhcp-server on the EDA-Appliance. We usually make the EDA DHCP-Master of a hidden-network which is only exposed to our ESX-Hosts and the EDA itself. Here they will get a IP from the EDA including the DHCP Options for PXE booting.

All that is left to do is restart a host (after you set maint mode, of course), select "boot from network" and then you need to select to appropriate profile os this host. After 5 more minutes your newly staged ESXi host is ready for work.

<del>We are currently in version 5.0.4 of the appliance but we still have lots of things on the roadmap. If there is a major update I will let you know and update this post.</del>

Update 2017: This appliance has been retired. I have written a complete new appliance and it is called it "Bechtle Steffen ES**X**i **S**taging **A**ppliance (xSA)", currently at version 1.4. Blog Post about the xSA: [Link](https://virtualfrog.wordpress.com/2017/05/02/introducing-the-new-esxi-staging-appliance/)
