---
layout: post
title: Backing up Fault-Tolerance protected VMs (the operation is not supported on
  the object)
date: 2017-07-25 11:16:44.000000000 +02:00
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
  _thumbnail_id: '1352'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:56:"https://twitter.com/virtual_dd/status/889776430942482432";}}
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '7467059220'
  _publicize_done_9340485: '1'
  _wpas_done_9383931: '1'
  publicize_twitter_user: virtual_dd
  publicize_linkedin_url: https://www.linkedin.com/updates?discuss=&scope=391645417&stype=M&topic=6295542128156114944&type=U&a=S_UM
  _publicize_done_14862392: '1'
  _wpas_done_14749148: '1'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2017/07/25/backing-up-fault-tolerance-protected-vms-the-operation-is-not-supported-on-the-object/"
excerpt: How to get rid of the error "the operation is not supported on the object"
---
One of my customers called me up and told me that their VMware Fault-Tolerance (FT) VMs can no longer be backed up. The error it throws says: " The operation is not supported on the object."

When I looked at the recent task&events I was able to see that the backup was successful up to a point where someone reconfigured FT on that VM. So I went into the documentation ([Link here](https://docs.vmware.com/en/VMware-vSphere/6.0/com.vmware.vsphere.avail.doc/GUID-F5264795-11DA-4242-B774-8C3450997033.html)) and saw that traditional VMware Snapshots are not supported on Fault Tolerance enabled VMs. There is a supported backup method for disk-only snapshots but that only works with the new FT (not legacy FT).

<!--more-->

New FT was introduced with vSphere 6.0. It enables the hypervisor to protect more than just single-core VMs, which many customers were looking for. Legacy FT is the way FT worked up until vSphere 5.5 where you could only protect a single CPU VMs.

So how did this customer get the message "The operation is not supported on the object" if they were running vSphere 6.0 Update 3? The answer is simple:

They used the C#-Client to do the configuration of FT!

This will create a "legacy Fault Tolerance" configuration for the VM. It makes sense if you read what the C#-Client Interface tells you just before you login: All new features after vSphere 5.1 are only available on the Web-Client. This includes the new FT introduced with vSphere 6.0.

The solution to get rid of the error: Reconfigure the VM for FT in the Web-Client and the backups worked again.

