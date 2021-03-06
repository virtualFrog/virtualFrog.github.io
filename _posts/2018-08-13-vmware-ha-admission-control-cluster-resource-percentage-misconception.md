---
layout: post
title: VMware HA Admission Control Cluster Resource Percentage Misconception
date: 2018-08-13 13:12:36.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- VCDX
- virtualFrog Posts
- VMware
tags: []
meta:
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:57:"https://twitter.com/virtual_dd/status/1028962547435753472";}}
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _thumbnail_id: '1352'
  timeline_notification: '1534158761'
  _publicize_job_id: '21033226341'
  _publicize_done_9340485: '1'
  _wpas_done_9383931: '1'
  publicize_twitter_user: virtual_dd
  publicize_linkedin_url: www.linkedin.com/updates?topic=6434728241012961281
  _publicize_done_14862392: '1'
  _wpas_done_14749148: '1'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2018/08/13/vmware-ha-admission-control-cluster-resource-percentage-misconception/"
---
Today I was working on my VCDX design and came across HA admission control. With VMware vSphere 6.5 a new feature introduced was "Cluster resource percentage". This setting would allow you to choose a number of host failures to tolerate and vSphere would then go ahead and calculate the percentage of that resource reservation. Easy, right? Not really, if you take a peak at the details of this configuration.<!--more-->

In my proposed design I wanted to use HA Admission Control to ensure that adequate resources are available for a site failover. I configured my HA Admission control to save 50% of the resources for the failover case. When I mentioned this in my VCDX prep group somebody asked me if I used resource reservations on my VMs. I did not, and did not want to for a couple of reasons.

Intrigued by his questioning I took a better look at the calculation that is happening for HA Admission Control Resource percentage setting. It turns out that the calculation ignores the configured amount of resources except when they are reserved. If that is not the case HA assumes that 32MhZ and 0 MB of RAM (plus the overhead of running this VM) will be sufficient per VM. With that low resource allocation assumption for the calculation the HA Admission Control feature is not working as I had intended for my design. In the worst case scenario it would allow a lot more VMs to be powered on that I had anticipated and not leaving me with enough resources to run the protected workloads in a disaster recovery case.

The colleague from my VCDX prep group who is also working on his VCDX submission told me that I should leave out HA Admission Control and only mention it as a fallback solution if resources were to be reserved.

Here are the links to the documentation regarding this:

[VMware HA Admission Control](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.avail.doc/GUID-53F6938C-96E5-4F67-9A6E-479F5A894571.html)

[VMware HA Admission Control - Cluster Resource Percentage](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.avail.doc/GUID-FAFEFEFF-56F7-4CDF-A682-FC3C62A29A95.html)

One possible workaround is to set the values for the minimum MhZ and MB of RAM to a higher setting. This could be achieved per cluster in the HA advanced settings with the following two settings:

> das.vmcpuinmhz  
> das.vmmemoryinmb

[Here are the specifications about these advanced settings.](https://kb.vmware.com/s/article/2033250)

I just wanted to share this lesson I have learned today. In essence: Review every configuration in the official documentation to make sure it works as you assumed it did. This will save you from embarrassing questions down the road.

