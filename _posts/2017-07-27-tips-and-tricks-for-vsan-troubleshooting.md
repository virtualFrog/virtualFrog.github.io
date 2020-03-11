---
layout: post
title: Tips and Tricks for vSAN troubleshooting
date: 2017-07-27 14:07:28.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- virtualFrog Posts
- VMware
- vSAN
tags:
- Troubleshooting
meta:
  _thumbnail_id: '437'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:56:"https://twitter.com/virtual_dd/status/890544167646056452";}}
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '7546986030'
  _publicize_done_9340485: '1'
  _wpas_done_9383931: '1'
  publicize_twitter_user: virtual_dd
  publicize_linkedin_url: https://www.linkedin.com/updates?discuss=&scope=391645417&stype=M&topic=6296309860690599936&type=U&a=aU02
  _publicize_done_14862392: '1'
  _wpas_done_14749148: '1'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2017/07/27/tips-and-tricks-for-vsan-troubleshooting/"
---
Today I attended a Web-Session from VMware with the topic vSAN troubleshooting tips and tricks and I wanted to share those tips and tricks with you.

## Common sense

Ensure that your hardware is on the VMware HCL  
- Disks / Controllers / Firmare / Drivers

Ensure that you have up-to-date backups (and test the restore process fully)

<!--more-->

## vSAN Overview

Cluster: 2-64 physical hosts  
Host: 1-5 disk groups  
Disk Group: 1 flash for cache, 1-7 flash or HDD devices for capacity

vSAN Objects:  
- VM Home, VM Swap, VMDK, Delta Disk, Memory Delta

Storage Policies:  
- Applied at per VM level or VMDK level  
- Define protection level & performance

Each object is made up of one or more components (depending on your storage policy)

[caption id="attachment\_464" align="alignnone" width="600"] ![Screen Shot 2017-07-27 at 13.12.56]({{ site.baseurl }}/assets/images/screen-shot-2017-07-27-at-13-12-56.png) C1 & C2 = Components, W = Witness[/caption]

Component states:

- Active - component accessible
- Absent - Inaccessible, but no explicit error codes sensed
  - host outage or maint mode with "ensure accessibility"
  - rebuild begins after 60 minute timeout
- Degraded - Inaccessible with error codes sensed
  - device failure
  - rebuild begins immediately
- Active - stale
  - In queue of objects to rebuild

&nbsp;

## vSAN Tools

- vRealize Ops / Log Insight
- ESXCLI
- RVC
- Health Check
- [vSAN Observer](https://kb.vmware.com/kb/2064240) (for performance issues)

### ESXCLI

**esxcli vsan** - gives the available namespaces  
_New in 6.6: debug & health_

**esxcli vsan helath cluster list** - Gives you an overview with the traffic light system (green, yellow, red) where you see all tests  
**esxcli vsan health cluster get -t "vSAN Disk Balance"** &nbsp;- get results of a test from above command  
**esxcli vsan health cluster get -t "vSAN object health"** - vSAN object health could mean serious problems if status is red. (Get UUID of object to track the problem with those objects)

**esxcli vsan debug** - gives available namespaces  
**esxcli vsan debug resync summary** - give information of current resync process  
**esxcli vsan debug object health summary get** - gives you an overview of your health  
**esxcli vsan debug object list |more** - Gives back all objects back including component states  
**esxcli vsan debug disk list** - gives you information about your disk and if they can keep up  
**esxcli vsan debug controller list** - gives information about your disk controllers (HCL information, Queue depth)

&nbsp;

### RVC (Ruby vSphere Console)

The RVC is preinstalled on all vCenter Server variants.

vsan.check\_state 0  
 ![Screen Shot 2017-07-27 at 13.28.44]({{ site.baseurl }}/assets/images/screen-shot-2017-07-27-at-13-28-44.png)

vsan.disks\_stats 0  
 ![Screen Shot 2017-07-27 at 13.29.08.png]({{ site.baseurl }}/assets/images/screen-shot-2017-07-27-at-13-29-08.png)

vsan.cluster\_info 0  
 ![Screen Shot 2017-07-27 at 13.29.53]({{ site.baseurl }}/assets/images/screen-shot-2017-07-27-at-13-29-53.png)

### Health UI

![Screen Shot 2017-07-27 at 13.30.33]({{ site.baseurl }}/assets/images/screen-shot-2017-07-27-at-13-30-33.png)

vSAN Disk balance  
 ![Screen Shot 2017-07-27 at 13.32.00]({{ site.baseurl }}/assets/images/screen-shot-2017-07-27-at-13-32-00.png)

## vSAN Health

**python /usr/lib/vmware-vpx/vsan-health/vsan-vc-health-status.py \> /tmp/vsan\_status.txt**

![Screen Shot 2017-07-27 at 13.34.56]({{ site.baseurl }}/assets/images/screen-shot-2017-07-27-at-13-34-56.png)

Running this on a individual node:  
Location: /usr/lib/vmware/vsan/bin/vsan-health-status.pyc

> python&nbsp;/usr/lib/vmware/vsan/bin/vsan-health-status.pyc

[KB 2107705](http://kb.vmware.com/kb/2107705) has more information on that.

When to use those: Health Service on vCenter not available

## Use Cases

Cluster with 6 hosts. Three nodes were added. No capacity added after adding the hosts.

> esxcli vsan storage list

![Screen Shot 2017-07-27 at 13.40.13]({{ site.baseurl }}/assets/images/screen-shot-2017-07-27-at-13-40-13.png) **In CMMDS: false - take note of the naa... ID**

- Check vobd.log (search for naa... ID)
- Check boot.gz (use zcat; search for naa... ID)
  - Disk naa... ID detected to be a snapshot

Resolution:  
- Disk cannot be added because it already has a filesystem on it  
- Disk was given UUID, it was in a cluster and used at some point  
- Verified to delete data  
- Used partedutil to kill partitions  
- Delete disk groups and recreate disk groups

&nbsp;

### Useful Logs:

> vobd.log

Search for "problem" and "permanent"

&nbsp;

