---
layout: post
title: 'PowerCLI: Find and disconnect CD Drives on your VMs'
date: 2017-07-31 12:42:03.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- PowerCLI
- Scripting
- virtualFrog Posts
- VMware
tags: []
meta:
  _thumbnail_id: '29'
  publicize_twitter_user: virtual_dd
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '7707573146'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:56:"https://twitter.com/virtual_dd/status/891972222625226753";}}
  _publicize_done_9340485: '1'
  _wpas_done_9383931: '1'
  publicize_linkedin_url: https://www.linkedin.com/updates?discuss=&scope=391645417&stype=M&topic=6297737914637914112&type=U&a=tyZV
  _publicize_done_14862392: '1'
  _wpas_done_14749148: '1'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2017/07/31/powercli-find-and-disconnect-cd-drives-on-your-vms/"
---
One of the things that regularly pops up in our health checks in VMware environments are connected CD Drives that prevent DRS from functioning properly or prevents a host from entering maintenance mode when doing thing like patching.

In large environments it can get very tedious to disconnect all those drives manually. With PowerCLI it is just one line.

<!--more-->

If you want to free up a host for maintenance mode you can do it with two lines:

[code language="powershell"]Get-VMHost \<your host\> |Get-VM | Where-Object {$\_.PowerState –eq “PoweredOn”} | Get-CDDrive | Set-CDDrive -NoMedia -Confirm:$False  
Get-VMHost \<your host\>| Set-VMHost -VMHost Host -State "Maintenance"[/code]

If you want to disconnect all of your CD-ROMs:  
[code language="powershell"]Get-VM | Where-Object {$\_.PowerState –eq “PoweredOn”} | Get-CDDrive | Set-CDDrive -NoMedia -Confirm:$False[/code]

If you just want to list all the VMs that have connected CD Drives:  
[code language="powershell"]Get-VM | Where-Object {$\_.PowerState –eq “PoweredOn”} | Get-CDDrive | FT Parent, IsoPath[/code]

