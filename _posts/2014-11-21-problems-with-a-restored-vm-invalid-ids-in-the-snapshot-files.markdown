---
author: virtualfrog
comments: true
date: 2014-11-21 14:16:44+00:00
layout: post
link: https://virtualfrog.wordpress.com/2014/11/21/problems-with-a-restored-vm-invalid-ids-in-the-snapshot-files/
slug: problems-with-a-restored-vm-invalid-ids-in-the-snapshot-files
title: 'Problems with a restored VM: Invalid IDs in the Snapshot Files'
wordpress_id: 15
categories:
- VMware
tags:
- Troubleshooting
---

Recently we've had the pleasure to restore a Database-VM from the most recent Backup because one of my coworkers accidentally deleted a vital .vmdk File on the datastore..

Well anyways, restrore was completed but threw an error-message saying that the snapshot could not be removed. Fine by me, right? So I went to the "Snapshot Manager" and deleted all the snapshots. The task went through perfectly but another message poppep up saying "VM need consolidation".
<!-- more -->
Okay, no problem. Over the GUI I tried to consolidate the disks, but that threw an error as well...

Time to connect to the ESXi host directly (over ssh) and look at the problem there. I went to the datastore containing the VM (well, actually the VM was spread across seven Datastores, but you get the point) and looked at the descriptor Files of the VMs.

It turned out that all the snapshot-descriptor-Files (somevm_1-000001.vmdk) had a wrong ParentCID. It referenced itself..

The next step was to replace all the wrong CIDs with the correct CID of the parent-disk. In a VM with 16 disks spread across 7 datastores this was quite a task. After I was finished (important: double-check that you put the corrent CID in the descriptor file, otherwise you could corrupt the entire VM!) all I had to do was to re-register the VM into the inventory. It then recognised the snapshot again, and the consolidate task went through without any errors.

That's it. Wrong CID in the descriptor files of the VM were resolved and we were able to strart the VM again.
