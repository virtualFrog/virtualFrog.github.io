---
layout: post
title: 'Problems with a restored VM: Invalid IDs in the Snapshot Files'
date: 2014-11-21 16:16:44.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- VMware
tags:
- Troubleshooting
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _wpas_skip_facebook: '1'
  _wpas_skip_google_plus: '1'
  _wpas_skip_linkedin: '1'
  _wpas_skip_tumblr: '1'
  _wpas_skip_path: '1'
  publicize_twitter_user: virtual_dd
  publicize_twitter_url: http://t.co/9Ue1ps3zad
  _wpas_done_9383931: '1'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:350746208;b:1;}}
  _thumbnail_id: '21'
  _edit_last: '75727371'
  geo_public: '0'
  _wpas_skip_9383931: '1'
  _wpas_skip_twitter: '1'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2014/11/21/problems-with-a-restored-vm-invalid-ids-in-the-snapshot-files/"
---
Recently we've had the pleasure to restore a Database-VM from the most recent Backup because one of my coworkers accidentally deleted a vital .vmdk File on the datastore..

Well anyways, restrore was completed but threw an error-message saying that the snapshot could not be removed. Fine by me, right? So I went to the "Snapshot Manager" and deleted all the snapshots. The task went through perfectly but another message poppep up saying "VM need consolidation".  
<!--more-->  
Okay, no problem. Over the GUI I tried to consolidate the disks, but that threw an error as well...

Time to connect to the ESXi host directly (over ssh) and look at the problem there. I went to the datastore containing the VM (well, actually the VM was spread across seven Datastores, but you get the point) and looked at the descriptor Files of the VMs.

It turned out that all the snapshot-descriptor-Files (somevm\_1-000001.vmdk) had a wrong ParentCID. It referenced itself..

The next step was to replace all the wrong CIDs with the correct CID of the parent-disk. In a VM with 16 disks spread across 7 datastores this was quite a task. After I was finished (important: double-check that you put the corrent CID in the descriptor file, otherwise you could corrupt the entire VM!) all I had to do was to re-register the VM into the inventory. It then recognised the snapshot again, and the consolidate task went through without any errors.

That's it. Wrong CID in the descriptor files of the VM were resolved and we were able to strart the VM again.

