---
layout: post
title: 'PowerCLI: Setting Resource Pools right'
date: 2019-01-09 11:42:12.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- PowerCLI
- Scripting
- vExpert
- virtualFrog Posts
- VMware
tags: []
meta:
  _thumbnail_id: '29'
  _wpas_done_9383931: '1'
  timeline_notification: '1547030536'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '26338060735'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:57:"https://twitter.com/virtual_dd/status/1082950687191908352";}}
  _publicize_done_9340485: '1'
  publicize_twitter_user: virtual_dd
  publicize_linkedin_url: www.linkedin.com/updates?topic=6488716378860769280
  _publicize_done_14862392: '1'
  _wpas_done_14749148: '1'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2019/01/09/powercli-setting-resource-pools-right/"
excerpt: I wrote a script to set resource pools right.
---
Much has been written about resources and resource pools. Especially in the recent deep dive books from Duncan Epping, Niels Hagoort and Frank Denneman as well as in many blogs from chris wahl and elasic sky.

I want to dedicate this post to resource pools because I still find them everywhere and they are not configured right.

<!--more-->

**Do not use resource pools as folders**

There are perfectly good folders in the “VMs and Templates” view for you to use. If you use resource pools as folders you can get hurt when a contention arises.

&nbsp;

**Do not use default share values&nbsp;“low”,&nbsp;“normal” and&nbsp;“high"**

Why? Because they do not account for the number of VMs or resources residing within. A resource pool is dynamic and so should the share number of that resource pool be.

Chris Wahl has posted a very nice picture about why not to do this back in 2012 and it sill holds true today:&nbsp;[https://wahlnetwork.com/2012/02/01/understanding-resource-pools-in-vmware-vsphere/](https://wahlnetwork.com/2012/02/01/understanding-resource-pools-in-vmware-vsphere/)

&nbsp;

**If you do use resource pools, place every VM in them**

Do not place VMs outside the resource pool (aka. the “resources” resource pool). What happens if you do? You make that VM a sibling to the other resource pools. So in a contention scenario this VM would get as much resources as a resource pool where potentially many more VMs reside.

&nbsp;

**What is the right way?**

PowerCLI to the rescue! This task screams to be automated and regularly run to always have your resource pools optimised. That is the only way to guarantee they will behave as expected in a contention scenario.

I have written a script before, but thanks to inspiration from this post ([https://www.elasticsky.de/en/2017/08/rebalance-your-resource-pools/](https://www.elasticsky.de/en/2017/08/rebalance-your-resource-pools/)) I have taken another look at my script and expanded it.

The script must be edited at lines 66 to 69. This is where you put your resource pools in and decide the factor by which their resources differ from each other:

![screenshot 2019-01-09 at 11.31.03]({{ site.baseurl }}/assets/images/screenshot-2019-01-09-at-11.31.03.png)

&nbsp;

Other than that you do not need to edit anything. I have taken care to include a synopsis for easy consumption but I have one dependency (aside from PowerCLI): PSLogging. That is a module from the Powershell Gallery that makes log files easier (at least for me). Here is the link to that module:&nbsp;[https://www.powershellgallery.com/packages/PSLogging](https://www.powershellgallery.com/packages/PSLogging)

And last but not least, my script can be found in my GitHub. [Directlink to the script](https://github.com/virtualFrog/PowerCLI-Scripts/blob/master/Set-ResourcePoolGranular.ps1). Please use at your own risk. And you can test the script by using the "Set-ResourcePool" command with the -WhatIf parameter (read the comments in the script).

And please feel free to improve upon it. Come to think of it, I should add a disconnect-vi server command at the end.

&nbsp;

