---
layout: post
title: 'PowerCLI: Releasing my vCheck Fork'
date: 2019-02-05 09:38:56.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- PowerCLI
- Scripting
- vExpert
- virtualFrog Posts
- VMware
tags: []
meta:
  _thumbnail_id: '29'
  timeline_notification: '1549355940'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '27297380301'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:57:"https://twitter.com/virtual_dd/status/1092704143167418369";}}
  _publicize_done_9340485: '1'
  _wpas_done_9383931: '1'
  publicize_twitter_user: virtual_dd
  publicize_linkedin_url: www.linkedin.com/updates?topic=6498469836098719744
  _publicize_done_14862392: '1'
  _wpas_done_14749148: '1'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2019/02/05/powercli-releasing-my-vcheck-fork/"
---
I have been working on a lot of improvements to the popular PowerCLI HTML Framework script ["vCheck Daily Report"](https://github.com/alanrenouf/vCheck-vSphere) which was originally developed by [Alan Renouf](https://twitter.com/alanrenouf)and has been enhanced by the [VMware Code Community](https://code.vmware.com/home) over the past few years.

Development of new features has slowed down quite a bit over the last year or so. Incidentally the original vCheck was the perfect starting point to build upon for an automated set of tests I wanted to perform on vSphere environment for our customers.

So I ended up cloning the original to our company internal GitLab and started working on improvements and added a lot of new plugins for the VCSA vCenter.

<!--more-->

Today I have decided to publish all of my work to my public [GitHub Page](https://github.com/virtualFrog). I have officially forked the original project and copied over all of my new plugins and changes. I have changed quite a lot and changed each and every plugin in at least the way it displays itself in the Table of Contents (ToC) of the HTML output.

The fork is of course running under the same license as the original and everyone is welcome to contribute or further fork my fork or merge (the good) changes back to the original. I will further work on enhancing this fork and will also try and implement the fixes of issues that go into the original project (I think I have all of them fixed that were published as of February 5th 2019).

If you have any questions about the fork please write a comment here or reach out to me on [twitter](https://twitter.com/virtual_frog).

[Link to my vCheck Daily Report Fork aka. vSphereChecker](https://github.com/virtualFrog/vCheck-vSphere)

Here is a picture of an output using the DarkClarity Theme:

![Screenshot 2019-02-05 at 09.34.54]({{ site.baseurl }}/assets/images/screenshot-2019-02-05-at-09.34.54.png)

