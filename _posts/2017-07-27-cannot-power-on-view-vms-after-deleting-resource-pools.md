---
layout: post
title: Cannot power on View VMs after deleting resource pools
date: 2017-07-27 10:29:29.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- virtualFrog Posts
- VMware
- VMware View
tags:
- Troubleshooting
meta:
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:56:"https://twitter.com/virtual_dd/status/890489311770050560";}}
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _thumbnail_id: '432'
  _publicize_job_id: '7541786731'
  _publicize_done_9340485: '1'
  _wpas_done_9383931: '1'
  publicize_twitter_user: virtual_dd
  publicize_linkedin_url: https://www.linkedin.com/updates?discuss=&scope=391645417&stype=M&topic=6296255008132329472&type=U&a=88tZ
  _publicize_done_14862392: '1'
  _wpas_done_14749148: '1'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2017/07/27/cannot-power-on-view-vms-after-deleting-resource-pools/"
---
One of my colleagues had a problem the other night. He was called to troubleshoot a VMware View environment which was not running as expected. One thing he did during the cleaning up was to get rid of a number of resource pools that were only created for the purpose of putting the VMs into "folders" when viewed from the old C# Client.

<!--more-->

After fixing other errors he was ready to create new linked clones. But they could not power on. The error message said:

> The available memory resources in the parent resource pool are insufficient for this operation

Turns out the hosts had a file in /etc/vmware which referred to the resource pools and was not properly cleaned up. In order to solve this problem the following had to be run on each esxi host:

> rm /etc/vmware/pools.xml  
> /etc/init.d/hostd restart  
> /etc/init.d/vpxa restart

After that the VMs were able to power-on as expected and the environment was up and running again.

