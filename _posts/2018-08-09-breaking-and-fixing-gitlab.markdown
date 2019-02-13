---
author: virtualfrog
comments: true
date: 2018-08-09 17:52:42+00:00
layout: post
link: https://virtualfrog.wordpress.com/2018/08/09/breaking-and-fixing-gitlab/
slug: breaking-and-fixing-gitlab
title: Breaking and Fixing GitLab
wordpress_id: 787
categories:
- General
---

Today I realized that my GitLab server was severely out of date and the underlying ubuntu (17.10) was end-of-life. So I thought it would be an easy fix to upgrade the GitLab community edition and then do a distribution upgrade to get to the latest ubuntu (18.04) build.

How wrong I was.<!-- more -->The problems began after issuing the "sudo apt-get upgrade" command. The upgrader told me that my current gitlab version did not allow a direct upgrade to 11.1.4 because it was not the latest 10.x version. So I should first upgrade to version 10.8 and then try to upgrade to 11.4.1.

Okay, a quick google query landed me on the download page of the gitlab versions corresponding to the many different linux flavors out there. [Link to the gitlab download page for the community edition](https://packages.gitlab.com/gitlab/gitlab-ce).

After a quick search I found the latest 10.8 build for ubuntu/xenial and the page offered me the wget command:

![Screen Shot 2018-08-09 at 19.25.32](https://virtualfrog.files.wordpress.com/2018/08/screen-shot-2018-08-09-at-19-25-32.png)

After getting the package down unto my server I was able to upgrade gitlab to the latest 10.8 version using the command "sudo dpgk -i [path to the downloaded file]".

Great, first step achieved, GitLab is now 10.8 and is still working.

Next step was to issue the "sudo apt-get upgrade" command again and this time it actually achieved the upgrade to 11.4.1. But then my GitLab was not reachable anymore. I had no idea why. My first thought was, okay I am running on a no longer supported ubuntu version so I decided to upgrade to version 18.04 and hopefully my problem would fix itself.

The command "sudo do-distribution-upgrade" went through but could not clean up all of my old linux kernels from previous upgrades. Due to the fact that my gitlab was still not working as expected and I had lots of leftovers from the previous installations of ubuntu I decided to build a new ubuntu VM.

Downloading the latest server image from the official [downloadpage](https://www.ubuntu.com/download/server) was done very quickly and I deployed a new VM. I'm sparing you the details of the installation of the new ubuntu server because it is very easy, just follow the instructions.

I did of course want to save the data from my previous GitLab server. The following command will backup everything:

"sudo gitlab-rake gitlab:backup:create"

Complete instructions [here](https://docs.gitlab.com/ee/raketasks/backup_restore.html).

After completing the setup of gitlab following the instruction from [here](https://about.gitlab.com/installation/#ubuntu) (You need to replace gitlab-ee with gitlab-ce) I was ready to restore the previous backup. I copied the data over to the new server and followed the instructions [here](https://docs.gitlab.com/ee/raketasks/backup_restore.html#restore) in order to complete the restore.

Unfortunately my GitLab server was still not responding to HTTPS requests.

After some wasted hours I stumbled upon the log:

/var/log/gitlab/nginx/current

This had entries that finally told me that my certificates were not in the right places on the new system. Following the log messages I was able to successfully restore gitlab to working condition.

Just for the completeness will paste some error messages from the log I mentioned (maybe other people will find them using google someday):


<blockquote>2018-08-09_17:07:53.24790 nginx: [emerg] BIO_new_file("/etc/gitlab/ssl/HOSTNAME.crt") failed (SSL: error:02001002:system library:fopen:No such file or directory:fopen('/etc/gitlab/ssl/HOSTNAME.crt','r') error:2006D080:BIO routines:BIO_new_file:no such file)

2018-08-09_17:11:05.37776 nginx: [emerg] SSL_CTX_use_PrivateKey_file("/etc/gitlab/ssl/HOSTNAME.key") failed (SSL: error:02001002:system library:fopen:No such file or directory:fopen('/etc/gitlab/ssl/HOSTNAME.key','r') error:20074002:BIO routines:FILE_CTRL:system lib error:140B0002:SSL routines:SSL_CTX_use_PrivateKey_file:system lib)</blockquote>


After some reflecting and further research I now know why this upgrade was so difficult. GitLab has introduced "Letsencrypt" functions into it which were then enabled by default. This lead to the system trying to replace my certificates with letsencrypt certificates. But it failed to do them properly and so I ended up with a system with corrupt certificates.

Now I'm enjoying the successful upgrade with my glorious login screen:

![Screen Shot 2018-08-09 at 19.50.07](https://virtualfrog.files.wordpress.com/2018/08/screen-shot-2018-08-09-at-19-50-07.png)
