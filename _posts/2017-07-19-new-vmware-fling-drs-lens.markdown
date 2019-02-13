---
author: virtualfrog
comments: true
date: 2017-07-19 07:44:02+00:00
layout: post
link: https://virtualfrog.wordpress.com/2017/07/19/new-vmware-fling-drs-lens/
slug: new-vmware-fling-drs-lens
title: 'New VMware Fling: DRS Lens'
wordpress_id: 161
categories:
- General
- VMware
---

I have always been a fan of the VMware Flings. There have been some great Flings in the past that eventually made it into VMware's products.

The most recent Fling I looked at is the "DRS Lens" which was released a couple of days ago. Yesterday I took the time to deploy the OVA in our IT-Lab and thought I'd share the results.<!-- more -->


## Installation


The installation is very straight forwards. All you have to do is define a IP Pool in your vCenter. (follow this link to know how to do it: [Link](https://docs.vmware.com/en/VMware-vSphere/6.0/com.vmware.vsphere.hostclient.doc/GUID-5B3AF10D-8E4A-403C-B6D1-91D9171A3371.html))

So basically you just deploy the OVA using the import wizard and define a IP for the appliance. In my case the first boot was very slow, I could not tell what it was doing exactly but it took quite a while to come online.


## First Step: Add your vCenter Server


When the deployment completes you can either access the appliance over HTTP or HTTPS:

http://<your-ip>:8080/drs/app
https://<your-ip>/drs/app

The first thing it will ask you is to point it to a vCenter. If you have multiple vCenter server you need to deploy multiple instances because I could not find a way to add more than one vCenter (version 1.0).

After that you configure the metrics that determine wether your VMs are happy or not. This is a swap and cpu wait metrix. I just used the defaults and moved on.

Next you will have to start the monitoring (and input credentials again) and configure for how long the monitoring should be active. For demo purposes I only chose 1 hour.


## Enjoy insight into DRS operations of your cluster


![Screen Shot 2017-07-19 at 09.20.49](https://virtualfrog.files.wordpress.com/2017/07/screen-shot-2017-07-19-at-09-20-49.png)

on the "happiness" tab you can see if your vms were happy or not.

![Screen Shot 2017-07-19 at 09.21.06](https://virtualfrog.files.wordpress.com/2017/07/screen-shot-2017-07-19-at-09-21-06.png)

here you can see the threshold of your drs cluster. during this test I changed drs to applying all recommendations (so I can see some action), this has caused the "tolerable inbalance" threshold to drop.

![Screen Shot 2017-07-19 at 09.21.17](https://virtualfrog.files.wordpress.com/2017/07/screen-shot-2017-07-19-at-09-21-17.png)

as a result of my cluster settings change you can see the vmotions that have been triggered by drs.

![Screen Shot 2017-07-19 at 09.21.28](https://virtualfrog.files.wordpress.com/2017/07/screen-shot-2017-07-19-at-09-21-28.png)

on the operations tab you have an overview of all the changes in your cluster (drs settings change, vmotions, vSAN stuff)


## Conclusion


After playing with it for a little while I have to say I like the additional insights this fling provides. One use-case I can already see is that some customers don't want drs to move vms so much and with this fling they can monitor how many vmotions actually occur.

If you want to play with it as well feel free to download the fling from [here](https://labs.vmware.com/flings/drs-lens)

Update: Today they released version 1.1! According to the changelog you can now use 5.5 vCenters as well. Nice job. I actually tested on vSphere 6.0 Update 3 so I'll wait for the next release to update.
