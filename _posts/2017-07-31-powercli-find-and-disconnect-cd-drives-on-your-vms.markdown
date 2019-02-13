---
author: virtualfrog
comments: true
date: 2017-07-31 10:42:03+00:00
layout: post
link: https://virtualfrog.wordpress.com/2017/07/31/powercli-find-and-disconnect-cd-drives-on-your-vms/
slug: powercli-find-and-disconnect-cd-drives-on-your-vms
title: 'PowerCLI: Find and disconnect CD Drives on your VMs'
wordpress_id: 623
categories:
- General
- PowerCLI
- Scripting
- VMware
---

One of the things that regularly pops up in our health checks in VMware environments are connected CD Drives that prevent DRS from functioning properly or prevents a host from entering maintenance mode when doing thing like patching.

In large environments it can get very tedious to disconnect all those drives manually. With PowerCLI it is just one line.

<!-- more -->

If you want to free up a host for maintenance mode you can do it with two lines:

[code language="powershell"]Get-VMHost <your host> |Get-VM | Where-Object {$_.PowerState –eq “PoweredOn”} | Get-CDDrive | Set-CDDrive -NoMedia -Confirm:$False
Get-VMHost <your host>| Set-VMHost -VMHost Host -State "Maintenance"[/code]

If you want to disconnect all of your CD-ROMs:
[code language="powershell"]Get-VM | Where-Object {$_.PowerState –eq “PoweredOn”} | Get-CDDrive | Set-CDDrive -NoMedia -Confirm:$False[/code]

If you just want to list all the VMs that have connected CD Drives:
[code language="powershell"]Get-VM | Where-Object {$_.PowerState –eq “PoweredOn”} | Get-CDDrive | FT Parent, IsoPath[/code]



