---
author: virtualfrog
comments: true
date: 2019-02-05 08:38:56+00:00
layout: post
link: https://virtualfrog.wordpress.com/2019/02/05/powercli-releasing-my-vcheck-fork/
slug: powercli-releasing-my-vcheck-fork
title: 'PowerCLI: Releasing my vCheck Fork'
wordpress_id: 1183
categories:
- General
- PowerCLI
- Scripting
- vExpert
- VMware
---

I have been working on a lot of improvements to the popular PowerCLI HTML Framework script ["vCheck Daily Report"](https://github.com/alanrenouf/vCheck-vSphere) which was originally developed by [Alan Renouf ](https://twitter.com/alanrenouf)and has been enhanced by the [VMware Code Community](https://code.vmware.com/home) over the past few years.

Development of new features has slowed down quite a bit over the last year or so. Incidentally the original vCheck was the perfect starting point to build upon for an automated set of tests I wanted to perform on vSphere environment for our customers.

So I ended up cloning the original to our company internal GitLab and started working on improvements and added a lot of new plugins for the VCSA vCenter.

<!-- more -->

Today I have decided to publish all of my work to my public [GitHub Page](https://github.com/virtualFrog). I have officially forked the original project and copied over all of my new plugins and changes. I have changed quite a lot and changed each and every plugin in at least the way it displays itself in the Table of Contents (ToC) of the HTML output.

The fork is of course running under the same license as the original and everyone is welcome to contribute or further fork my fork or merge (the good) changes back to the original. I will further work on enhancing this fork and will also try and implement the fixes of issues that go into the original project (I think I have all of them fixed that were published as of February 5th 2019).

If you have any questions about the fork please write a comment here or reach out to me on [twitter](https://twitter.com/virtual_frog).

[Link to my vCheck Daily Report Fork aka. vSphereChecker](https://github.com/virtualFrog/vCheck-vSphere)

Here is a picture of an output using the DarkClarity Theme:

![Screenshot 2019-02-05 at 09.34.54](https://virtualfrog.files.wordpress.com/2019/02/screenshot-2019-02-05-at-09.34.54.png)
