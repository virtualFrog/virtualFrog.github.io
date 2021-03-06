---
layout: post
title: 'New VMware Fling: DRS Lens'
date: 2017-07-19 09:44:02.000000000 +02:00
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
  _oembed_42d7eef99610086de9aa3c09c70030d9: "{{unknown}}"
  _oembed_07f391550cd9c785428023330ca9bb66: "{{unknown}}"
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '7243181605'
  _oembed_0cd5be318a7b3fb269d3fdc60cd3a5ac: "{{unknown}}"
  _oembed_e6781478706fe863ff3fd5cbc83d0e31: "{{unknown}}"
  _oembed_af6cb8bcddefb980257d290f014d7b4c: "{{unknown}}"
  _oembed_4d232d5c648db402de1cbf26de65f76a: "{{unknown}}"
  _oembed_4e1bbbf23589a12a9d1a50679b1161d2: "{{unknown}}"
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:56:"https://twitter.com/virtual_dd/status/887578773284995072";}}
  _publicize_done_9340485: '1'
  _wpas_done_9383931: '1'
  publicize_twitter_user: virtual_dd
  _oembed_12178896fbf39f2d524600453e0b40a8: "{{unknown}}"
  _oembed_596ea20d428abf969a79b01558e87162: "{{unknown}}"
  _oembed_7e4ce9e1d484cf9bd362f5f5a61cac56: "{{unknown}}"
  _oembed_8a87e3b19e2c370c827b7c91e1bad923: "{{unknown}}"
  _oembed_4660d233a8d1f9cf68011e4b0307f8b4: "{{unknown}}"
  _oembed_1d40963fb27105c60a0b5a6ea36bc2c2: "{{unknown}}"
  _oembed_d66d03dd2d9ac3332f66b03da0316612: "{{unknown}}"
  _oembed_ad469c8eca03a5ca0f982e06c557319c: "{{unknown}}"
  _oembed_2066549059b0997bb0c8709a883ca682: "{{unknown}}"
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2017/07/19/new-vmware-fling-drs-lens/"
---
I have always been a fan of the VMware Flings. There have been some great Flings in the past that eventually made it into VMware's products.

The most recent Fling I looked at is the "DRS Lens" which was released a couple of days ago. Yesterday I took the time to deploy the OVA in our IT-Lab and thought I'd share the results.<!--more-->

## Installation

The installation is very straight forwards. All you have to do is define a IP Pool in your vCenter. (follow this link to know how to do it: [Link](https://docs.vmware.com/en/VMware-vSphere/6.0/com.vmware.vsphere.hostclient.doc/GUID-5B3AF10D-8E4A-403C-B6D1-91D9171A3371.html))

So basically you just deploy the OVA using the import wizard and define a IP for the appliance. In my case the first boot was very slow, I could not tell what it was doing exactly but it took quite a while to come online.

## First Step: Add your vCenter Server

When the deployment completes you can either access the appliance over HTTP or HTTPS:

http://\<your-ip\>:8080/drs/app  
https://\<your-ip\>/drs/app

The first thing it will ask you is to point it to a vCenter. If you have multiple vCenter server you need to deploy multiple instances because I could not find a way to add more than one vCenter (version 1.0).

After that you configure the metrics that determine wether your VMs are happy or not. This is a swap and cpu wait metrix. I just used the defaults and moved on.

Next you will have to start the monitoring (and input credentials again) and configure for how long the monitoring should be active. For demo purposes I only chose 1 hour.

## Enjoy insight into DRS operations of your cluster

![Screen Shot 2017-07-19 at 09.20.49]({{ site.baseurl }}/assets/images/screen-shot-2017-07-19-at-09-20-49.png)

on the "happiness" tab you can see if your vms were happy or not.

![Screen Shot 2017-07-19 at 09.21.06]({{ site.baseurl }}/assets/images/screen-shot-2017-07-19-at-09-21-06.png)

here you can see the threshold of your drs cluster. during this test I changed drs to applying all recommendations (so I can see some action), this has caused the "tolerable inbalance" threshold to drop.

![Screen Shot 2017-07-19 at 09.21.17]({{ site.baseurl }}/assets/images/screen-shot-2017-07-19-at-09-21-17.png)

as a result of my cluster settings change you can see the vmotions that have been triggered by drs.

![Screen Shot 2017-07-19 at 09.21.28]({{ site.baseurl }}/assets/images/screen-shot-2017-07-19-at-09-21-28.png)

on the operations tab you have an overview of all the changes in your cluster (drs settings change, vmotions, vSAN stuff)

## Conclusion

After playing with it for a little while I have to say I like the additional insights this fling provides. One use-case I can already see is that some customers don't want drs to move vms so much and with this fling they can monitor how many vmotions actually occur.

If you want to play with it as well feel free to download the fling from [here](https://labs.vmware.com/flings/drs-lens)

Update: Today they released version 1.1! According to the changelog you can now use 5.5 vCenters as well. Nice job. I actually tested on vSphere 6.0 Update 3 so I'll wait for the next release to update.

