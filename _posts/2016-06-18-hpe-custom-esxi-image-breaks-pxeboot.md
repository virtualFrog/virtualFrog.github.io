---
layout: post
title: HPE Custom ESXi Image breaks pxeboot
date: 2016-06-18 13:43:27.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Kickstart
- Scripting
- virtualFrog Posts
tags: []
meta:
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:56:"https://twitter.com/virtual_dd/status/744133424516993024";}}
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '23954276395'
  _publicize_done_9340485: '1'
  _wpas_done_9383931: '1'
  publicize_twitter_user: virtual_dd
  publicize_linkedin_url: https://www.linkedin.com/updates?discuss=&scope=391645417&stype=M&topic=6149899116341137408&type=U&a=bMGM
  _publicize_done_14862392: '1'
  _wpas_done_14749148: '1'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2016/06/18/hpe-custom-esxi-image-breaks-pxeboot/"
---
With the ~~latest~~ &nbsp;vSphere 6.0 Update 2 May 2016 HPE Custom ESXi Hypervisor Image to download from VMware HPE managed to reference a uppercase letter in one of its configuration files.<!--more-->

This isn't a problem if you are mounting the image directly to the host you're trying to install. however, if you're like me and have a solution in place where the iso is stored centrally and gets streamed to the esxi hosts via pxeboot then you'll have a problem.

In my appliance the iso gets mounted and then the files are streamed from the (read-only) mountpoint to the esxi host.

The HPE Custom image does not come with any extensions like Juliet or Rock (which basically allows a linux/windows box to understand upper and lowercase letters when mounting a iso9660 file). So when I mount this iso in my appliance the configuration file references a file called "amsHelpe.v00" but due to the iso being mounted in all lowercase the file is actually named "amshelpe.v00".

We've reached out to HPE Engineering and told them about this problem. They will address it in the next release of HPE custom images.

Until then the only workaround is: copy the contents of your iso instead of mounting it and then go in and rename that one file.

Update: The issue was fixed in the next upgrade. Current images don't have this error anymore.

