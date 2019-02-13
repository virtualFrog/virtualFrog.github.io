---
author: virtualfrog
comments: true
date: 2017-07-18 12:30:02+00:00
layout: post
link: https://virtualfrog.wordpress.com/2017/07/18/my-solution-to-share-powershell-scripts/
slug: my-solution-to-share-powershell-scripts
title: my solution to share powershell scripts
wordpress_id: 125
categories:
- General
---

If you work in the field of VMware you tend to automate things as soon as the opportunity presents itself. Either to save time or to rid yourself of tedious work that repeats itself over and over.<!-- more -->

So over the years I have written many PowerCLI scripts to automate one thing or another and saved it somewhere on my disk. When a co-worker asked me about some things he wanted to automate I always had done something similar in the past and wanted to share it with him. Exchange filters prohibit sending .ps1 file so we had to copy past in txt form and send over e-mail. This is a very outdated approach.

A couple of months ago I got sick of this an was looking for a platform to share such things. I could not do that publicly (because scripts I write on company time are company property) so I was looking for a solution to have on-premise.

Enter GitLab Community Edition (https://about.gitlab.com/ ). It was very easy to install and configure. (I'm not going to write about the details of the installation, because it depends which underlaying OS you want to use, and the site itself has good instructions).

We now have a semi-public (user access management) GitLab which all of our engineers can access on the road and download the repositories to their laptops.

This way they can update older scripts, add new scripts and in general benefit from the work our team does everyday without having the reinvent the wheel every time a new requirement comes up.

I urge you to set up a GitLab for yourself, even if it is just you that works on it. The versioning and availability is just awesome when it comes to saving your hard work somewhere besides the hard disk (and some backups, of course).
