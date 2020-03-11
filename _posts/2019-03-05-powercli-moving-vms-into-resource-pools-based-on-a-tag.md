---
layout: post
title: 'PowerCLI: Moving VMs into resource pools based on a tag'
date: 2019-03-05 20:38:17.000000000 +01:00
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
tags: []
meta:
  _thumbnail_id: '29'
  _rest_api_client_id: "-1"
  timeline_notification: '1551814701'
  _rest_api_published: '1'
  _publicize_job_id: '28307985386'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:57:"https://twitter.com/virtual_dd/status/1103016930515083266";}}
  _publicize_done_9340485: '1'
  _wpas_done_9383931: '1'
  publicize_twitter_user: virtual_dd
  publicize_linkedin_url: ''
  _publicize_done_21519738: '1'
  _wpas_done_22130612: '1'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2019/03/05/powercli-moving-vms-into-resource-pools-based-on-a-tag/"
excerpt: A script which automates the assignment of VMs into resource pools based
  on tags. Has some added functionality to make sure all VMs have tags.
---
### Resource pools once again..

When I was helping a customer upgrade to vSphere 6.7 recently I came across another misuse of resource pools. After we finished the upgrade we discussed how to best use resource pools and how to automate as much as possible from this new process in order to make sure it is followed.

<!--more-->  
I wrote about resource pools [here](https://virtualfrog.wordpress.com/2019/01/09/powercli-setting-resource-pools-right/)&nbsp;and also provided [a script](https://github.com/virtualFrog/PowerCLI-Scripts/blob/master/Set-ResourcePoolGranular.ps1) which automates the calculation of the share values for each pool based on a factor that is defined. Like for example the "High" resource pool should get a factor of 8 while the "Low" resource pool has a factor of 1. This ensures that in time of contention the VMs in the "High" resource pool get 8 times more resources than the VMs in the "Low" resource pool.

After talking to many customers about resource pools this is actually the exact behaviour they were expecting when they set up their resource pools.

I usually tell my customers to use tags as often as possible. Tags are a great tool to ease automation of operation tasks. My recommendation is to use a tag category named "ResourcePool" which has a "Single" cardinality (which means only one tag from that category per object) and can only be applied to virtual machines.

### PowerCLI to the rescue

In PowerCLI this would be a simple one-liner:

[code language=powershell]New-TagCategory -Name "ResourcePool" -Cardinality "Single" -Description "Category for resource pool assignment" -Confirm:$false[/code]

In order to create tags for the newly created tag category one could use yet another simple one-liner:

[code language=powershell]New-Tag -Name "NameOfYourTag" -Category "ResourcePool" -Description "Tag for VMs to move into corresponding Resource pool" -Confirm:$false[/code]

Now that we have created a tag category and a set of tags it is up to the operations team of the customer to define which VM will receive which tag.

How do we go from there? We need to make sure that the tags are used for the resource pool assignment. And we also want to make sure that when we change a tag that the VM will be moved to the correct resource pool as well.

Instead of pasting the entire script in here I will just link to [my GitHub Repository](https://github.com/virtualFrog/PowerCLI-Scripts/blob/master/Move-VMsIntoResoucePoolsBasedOnTag.ps1).

The script is called: _[Move-VMsIntoResoucePoolsBasedOnTag.ps1](https://github.com/virtualFrog/PowerCLI-Scripts/blob/master/Move-VMsIntoResoucePoolsBasedOnTag.ps1)_

### The script

The script takes two parameters. The first one is "vCenter" which is basically self-explanatory. The second is the "defaultTag" and this one is optional. The "defaultTag" parameter will assign a tag with that name to all VMs that currently do not have a "ResourcePool" tag yet. This will ensure that all your VMs will first be tagged before they are moved into resource pools.

The TagCategory defaults to "ResourcePool". But this can of course be modified by changing the variable on line 63 in the script.

The script has a dependency on the [PSLogging](https://www.powershellgallery.com/packages/PSLogging/) module which I have used in some of my previous scripts as well. Thank to that module the script will generate a log with all very useful information that will make troubleshooting the script a breeze.

And if you need to schedule the script (which I recommend doing) you need to modify two lines (actually, uncomment two lines and comment two lines) to make that work. This is done on lines 75-80. You will need to create a VI-CredentialStoreItem, but I have provided the necessary command in the synopsis of the script.

To get the information from the script you can use the "get-help" cmdlet on it:

[code language=powershell]get-help ./Move-VMsIntoResourcePoolBasedOnTag.ps1[/code]

I tried to catch every error I could think of. The script will actually create tags, the category and the corresponding resource pools if they do not exist. If you do find an error, please let me know or submit a pull-request or issue on my github. That would be much appreciated.

And by the way, this script is meant as a companion script to [my previous script about optimizing resource pools](https://github.com/virtualFrog/PowerCLI-Scripts/blob/master/Set-ResourcePoolGranular.ps1). The script can be found on my [Github](https://github.com/virtualFrog/PowerCLI-Scripts/blob/master/Set-ResourcePoolGranular.ps1) and the [blog post](https://virtualfrog.wordpress.com/2019/01/09/powercli-setting-resource-pools-right/) about it is here.

