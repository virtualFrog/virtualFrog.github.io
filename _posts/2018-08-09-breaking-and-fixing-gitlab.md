---
layout: post
title: Breaking and Fixing GitLab
date: 2018-08-09 19:52:42.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- virtualFrog Posts
tags: []
meta:
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:57:"https://twitter.com/virtual_dd/status/1027613678097580034";}}
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _thumbnail_id: '335'
  _publicize_job_id: '20908295728'
  timeline_notification: '1533837166'
  _publicize_done_9340485: '1'
  _wpas_done_9383931: '1'
  publicize_twitter_user: virtual_dd
  publicize_linkedin_url: www.linkedin.com/updates?topic=6433379371964334080
  _publicize_done_14862392: '1'
  _wpas_done_14749148: '1'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2018/08/09/breaking-and-fixing-gitlab/"
---
Today I realized that my GitLab server was severely out of date and the underlying ubuntu (17.10) was end-of-life. So I thought it would be an easy fix to upgrade the GitLab community edition and then do a distribution upgrade to get to the latest ubuntu (18.04) build.

How wrong I was.<!--more-->The problems began after issuing the "sudo apt-get upgrade" command. The upgrader told me that my current gitlab version did not allow a direct upgrade to 11.1.4 because it was not the latest 10.x version. So I should first upgrade to version 10.8 and then try to upgrade to 11.4.1.

Okay, a quick google query landed me on the download page of the gitlab versions corresponding to the many different linux flavors out there. [Link to the gitlab download page for the community edition](https://packages.gitlab.com/gitlab/gitlab-ce).

After a quick search I found the latest 10.8 build for ubuntu/xenial and the page offered me the wget command:

![Screen Shot 2018-08-09 at 19.25.32]({{ site.baseurl }}/assets/images/screen-shot-2018-08-09-at-19-25-32.png)

After getting the package down unto my server I was able to upgrade gitlab to the latest 10.8 version using the command "sudo dpgk -i [path to the downloaded file]".

Great, first step achieved, GitLab is now 10.8 and is still working.

Next step was to issue the "sudo apt-get upgrade" command again and this time it actually achieved the upgrade to 11.4.1. But then my GitLab was not reachable anymore. I had no idea why. My first thought was, okay I am running on a no longer supported ubuntu version so I decided to upgrade to version 18.04 and hopefully my problem would fix itself.

The command "sudo do-distribution-upgrade" went through but could not clean up all of my old linux kernels from previous upgrades. Due to the fact that my gitlab was still not working as expected and I had lots of leftovers from the previous installations of ubuntu I decided to build a new ubuntu VM.

Downloading the latest server image from the official [downloadpage](https://www.ubuntu.com/download/server)&nbsp;was done very quickly and I deployed a new VM. I'm sparing you the details of the installation of the new ubuntu server because it is very easy, just follow the instructions.

I did of course want to save the data from my previous GitLab server. The following command will backup everything:

"sudo gitlab-rake gitlab:backup:create"

Complete instructions [here](https://docs.gitlab.com/ee/raketasks/backup_restore.html).

After completing the setup of gitlab following the instruction from [here](https://about.gitlab.com/installation/#ubuntu)&nbsp;(You need to replace gitlab-ee with gitlab-ce) I was ready to restore the previous backup. I copied the data over to the new server and followed the instructions [here](https://docs.gitlab.com/ee/raketasks/backup_restore.html#restore)&nbsp;in order to complete the restore.

Unfortunately my GitLab server was still not responding to HTTPS requests.

After some wasted hours I stumbled upon the log:

/var/log/gitlab/nginx/current

This had entries that finally told me that my certificates were not in the right places on the new system. Following the log messages I was able to successfully restore gitlab to working condition.

Just for the completeness will paste some error messages from the log I mentioned (maybe other people will find them using google someday):

> 2018-08-09\_17:07:53.24790 nginx: [emerg] BIO\_new\_file("/etc/gitlab/ssl/HOSTNAME.crt") failed (SSL: error:02001002:system library:fopen:No such file or directory:fopen('/etc/gitlab/ssl/HOSTNAME.crt','r') error:2006D080:BIO routines:BIO\_new\_file:no such file)
> 
> 2018-08-09\_17:11:05.37776 nginx: [emerg] SSL\_CTX\_use\_PrivateKey\_file("/etc/gitlab/ssl/HOSTNAME.key") failed (SSL: error:02001002:system library:fopen:No such file or directory:fopen('/etc/gitlab/ssl/HOSTNAME.key','r') error:20074002:BIO routines:FILE\_CTRL:system lib error:140B0002:SSL routines:SSL\_CTX\_use\_PrivateKey\_file:system lib)

After some reflecting and further research I now know why this upgrade was so difficult. GitLab has introduced "Letsencrypt" functions into it which were then enabled by default. This lead to the system trying to replace my certificates with letsencrypt certificates. But it failed to do them properly and so I ended up with a system with corrupt certificates.

Now I'm enjoying the successful upgrade with my glorious login screen:

![Screen Shot 2018-08-09 at 19.50.07]({{ site.baseurl }}/assets/images/screen-shot-2018-08-09-at-19-50-07.png)

