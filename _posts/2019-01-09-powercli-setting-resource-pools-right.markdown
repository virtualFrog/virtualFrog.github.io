---
author: virtualfrog
comments: true
date: 2019-01-09 10:42:12+00:00
excerpt: I wrote a script to set resource pools right.
layout: post
link: https://virtualfrog.wordpress.com/2019/01/09/powercli-setting-resource-pools-right/
slug: powercli-setting-resource-pools-right
title: 'PowerCLI: Setting Resource Pools right'
wordpress_id: 1112
categories:
- General
- PowerCLI
- Scripting
- vExpert
- VMware
---

Much has been written about resources and resource pools. Especially in the recent deep dive books from Duncan Epping, Niels Hagoort and Frank Denneman as well as in many blogs from chris wahl and elasic sky.

I want to dedicate this post to resource pools because I still find them everywhere and they are not configured right.

<!-- more -->

**Do not use resource pools as folders**

There are perfectly good folders in the “VMs and Templates” view for you to use. If you use resource pools as folders you can get hurt when a contention arises.



**Do not use default share values “low”, “normal” and “high"**

Why? Because they do not account for the number of VMs or resources residing within. A resource pool is dynamic and so should the share number of that resource pool be.

Chris Wahl has posted a very nice picture about why not to do this back in 2012 and it sill holds true today: [https://wahlnetwork.com/2012/02/01/understanding-resource-pools-in-vmware-vsphere/](https://wahlnetwork.com/2012/02/01/understanding-resource-pools-in-vmware-vsphere/)



**If you do use resource pools, place every VM in them**

Do not place VMs outside the resource pool (aka. the “resources” resource pool). What happens if you do? You make that VM a sibling to the other resource pools. So in a contention scenario this VM would get as much resources as a resource pool where potentially many more VMs reside.



**What is the right way?**

PowerCLI to the rescue! This task screams to be automated and regularly run to always have your resource pools optimised. That is the only way to guarantee they will behave as expected in a contention scenario.

I have written a script before, but thanks to inspiration from this post ([https://www.elasticsky.de/en/2017/08/rebalance-your-resource-pools/](https://www.elasticsky.de/en/2017/08/rebalance-your-resource-pools/)) I have taken another look at my script and expanded it.

The script must be edited at lines 66 to 69. This is where you put your resource pools in and decide the factor by which their resources differ from each other:

![screenshot 2019-01-09 at 11.31.03](https://virtualfrog.files.wordpress.com/2019/01/screenshot-2019-01-09-at-11.31.03.png)



Other than that you do not need to edit anything. I have taken care to include a synopsis for easy consumption but I have one dependency (aside from PowerCLI): PSLogging. That is a module from the Powershell Gallery that makes log files easier (at least for me). Here is the link to that module: [https://www.powershellgallery.com/packages/PSLogging](https://www.powershellgallery.com/packages/PSLogging)

And last but not least, my script can be found in my GitHub. [Directlink to the script](https://github.com/virtualFrog/PowerCLI-Scripts/blob/master/Set-ResourcePoolGranular.ps1). Please use at your own risk. And you can test the script by using the "Set-ResourcePool" command with the -WhatIf parameter (read the comments in the script).

And please feel free to improve upon it. Come to think of it, I should add a disconnect-vi server command at the end.


