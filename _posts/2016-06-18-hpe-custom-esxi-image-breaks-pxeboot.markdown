---
author: virtualfrog
comments: true
date: 2016-06-18 11:43:27+00:00
layout: post
link: https://virtualfrog.wordpress.com/2016/06/18/hpe-custom-esxi-image-breaks-pxeboot/
slug: hpe-custom-esxi-image-breaks-pxeboot
title: HPE Custom ESXi Image breaks pxeboot
wordpress_id: 81
categories:
- General
- Kickstart
- Scripting
---

With the <del>latest</del> vSphere 6.0 Update 2 May 2016 HPE Custom ESXi Hypervisor Image to download from VMware HPE managed to reference a uppercase letter in one of its configuration files.<!-- more -->

This isn't a problem if you are mounting the image directly to the host you're trying to install. however, if you're like me and have a solution in place where the iso is stored centrally and gets streamed to the esxi hosts via pxeboot then you'll have a problem.

In my appliance the iso gets mounted and then the files are streamed from the (read-only) mountpoint to the esxi host.

The HPE Custom image does not come with any extensions like Juliet or Rock (which basically allows a linux/windows box to understand upper and lowercase letters when mounting a iso9660 file). So when I mount this iso in my appliance the configuration file references a file called "amsHelpe.v00" but due to the iso being mounted in all lowercase the file is actually named "amshelpe.v00".

We've reached out to HPE Engineering and told them about this problem. They will address it in the next release of HPE custom images.

Until then the only workaround is: copy the contents of your iso instead of mounting it and then go in and rename that one file.

Update: The issue was fixed in the next upgrade. Current images don't have this error anymore.
