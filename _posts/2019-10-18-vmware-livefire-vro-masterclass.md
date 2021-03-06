---
layout: post
title: 'VMware Livefire: vRO Masterclass'
date: 2019-10-18 15:48:04.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- vExpert
- virtualFrog Posts
- VMware
- vRealize Automation
- vRealize Orchestrator
tags:
- LiveFire
- Training
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  publicize_linkedin_url: ''
  _thumbnail_id: '1284'
  _publicize_done_9340485: '1'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:57:"https://twitter.com/virtual_dd/status/1185190842698534913";}}
  timeline_notification: '1571406490'
  _publicize_job_id: '36480139097'
  _wpas_done_22130612: '1'
  _publicize_done_21519738: '1'
  publicize_twitter_user: virtual_dd
  _wpas_done_9383931: '1'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2019/10/18/vmware-livefire-vro-masterclass/"
excerpt: This post is about my experience from this week's VMware Livefire training
  about vRealize Orchestrator that took place at the VMware offices in Zurich.
---
This week I had the pleasure to attend a special training at the VMware offices in Zurich. The class was all about the orchestrating product of VMware: vRealize Orchestrator.

![Nametag LiveFire Training]({{ site.baseurl }}/assets/images/livefirenametag.png "liveFireNameTag.png")

Even though we at BUSINESS IT AG consist of a small team we have had 3 of our engineers attending the class.

**Day 1: vRO Core Architecture**

The first day was all about the core Architecture of vRealize Orchestrator. Here are some things that I picked up from that first day.

- JavaScript is still being used inside vRO because in most modern technologies JavaScript is dominant (AngularJS, Typescript, etc.)
- The file format that one gets from most modern REST APIs ist JSON which translates directly to JavaScript objects
- The JavaScript is being parsed to Java using Rhino 1.7 engine. This is still in use today to ensure that workflows from ten years ago would still work today
- There are different “versions” of the Orchestrator that you get depending on your license. A foundation/essential vCenter license will only give you the “player” version of the orchestrator. You can only run but not design workflows with that. Actually for all vCenter Licenses you will get a limited vRealize Orchestrator version. The “full” version is bundled with Automation for example.
- The next version of vRO will probably retire the “legacy” java client and will focus on the html5 client that is already inside the current 7.6 version.

The rest of the time was filled with very technical explanations about the different objects one can use inside vRO. We also were able to complete the first labs. What I really liked about these labs is that it gives you a challenge. You can try to achieve the lab’s goal using your wits only but if you should get stuck you can display the solution.

The resources used in this class are available to everybody. If you are interested in completing the labs and getting the slides (all except the NDA content related slides of course) please follow this [link](https://vro-labs.livefire.solutions).

**Day 2: Deep dive into vRO things**

On the second day we started out with dynamic Types for vRO. Dynamic types is essentially how you create a plugin with JavaScript. There are a lot of steps to take in order to have this working correctly. If you want to follow those steps please click on the link above where all the labs are listed.

Further into the day our teacher told the story of how he created a vRO package to work with Microsoft DNS. He was actually able to mirror the DNS inventory tree into the vRO inventory and created the necessary actions to be able to create DNS-as-a-Service. This is very useful for us and I think I will try and implement this in our MEP (Multi-Cloud Experience Platform). If you want to be able to do this as well follow [this link](https://www.vcoteam.info/articles/learn-vco/323-how-to-create-a-microsoft-dns-dynamic-types-plug-in.html) to read all about it.

**Day 3: Wrapping things up**

As with most trainings that I have done in the past with VMware the last day is usually the day where everyone needs to leave early to catch flights back home. On this day we wrapped up the remaining sessions that we hadn’t covered up until now and then got a lot of time to work on the labs. The cool thing about this Masterclass is that the lab guides are publicly available (see the link above) and everyone can follow them without having access to the labs from the class. I will follow those labs in the coming months whenever time permits to catch up on some very neat techniques when it comes to vRealize Orchestrator.

All in all this was a very informative masterclass. As always the Livefire training do not disappoint and I can recommend this class to everyone interested in automation. My key take away from this class was that investing time in learning how to develop workflows in vRealize Orchestrator is time well spent and there is so much still to learn. I am looking forward to the next release (8.0) or orchestrator that will push for the web client and expose some more REST API goodness.

