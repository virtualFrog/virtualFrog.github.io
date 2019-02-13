---
author: virtualfrog
comments: true
date: 2017-07-27 08:29:29+00:00
layout: post
link: https://virtualfrog.wordpress.com/2017/07/27/cannot-power-on-view-vms-after-deleting-resource-pools/
slug: cannot-power-on-view-vms-after-deleting-resource-pools
title: Cannot power on View VMs after deleting resource pools
wordpress_id: 420
categories:
- General
- VMware
- VMware View
tags:
- Troubleshooting
---

One of my colleagues had a problem the other night. He was called to troubleshoot a VMware View environment which was not running as expected. One thing he did during the cleaning up was to get rid of a number of resource pools that were only created for the purpose of putting the VMs into "folders" when viewed from the old C# Client.

<!-- more -->

After fixing other errors he was ready to create new linked clones. But they could not power on. The error message said:


<blockquote>The available memory resources in the parent resource pool are insufficient for this operation</blockquote>


Turns out the hosts had a file in /etc/vmware which referred to the resource pools and was not properly cleaned up. In order to solve this problem the following had to be run on each esxi host:


<blockquote>rm /etc/vmware/pools.xml
/etc/init.d/hostd restart
/etc/init.d/vpxa restart</blockquote>


After that the VMs were able to power-on as expected and the environment was up and running again.
