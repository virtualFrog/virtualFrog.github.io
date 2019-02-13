---
author: virtualfrog
comments: true
date: 2016-06-18 11:16:48+00:00
layout: post
link: https://virtualfrog.wordpress.com/2016/06/18/vmware-sddc-livefire-training/
slug: vmware-sddc-livefire-training
title: VMware SDDC LiveFire Training
wordpress_id: 61
categories:
- VMware
tags:
- LiveFire
- Training
---

This week I was fortunate to be able to attend a LiveFire Class at the VMware UK Headquarters in Staines upon Thames in England.

The LiveFire SDDC class enables you to see the entire vRealize Suite working together in your own lab. The guys from VMware are very competent in their respective fields and were able to answer all of the attendee's questions.<!-- more -->

**Day 1: vRealize Automations Installation**

On the first day we were greeted by some very lovely VMware folks who invited us for coffee while we waited for our visitor badges. The headquarter is located at Flow 1 in Staines upon Thames and it is a very nice building.

The first thing on the agenda was the "meet and greet". So all attendees told the class who they were, what fields they were an expert in and one or two anecdotes from their personal lives. We've had mostly VMware Partners attending. They came from the Netherlands, England, Czechoslovakia , Hawaii, Poland, Dubai, Austria, France, and some other countries I forgot. I was the only one attending from Switzerland. After that the LiveFire team did the same. There was aemon (vRA), Hans (NSX), Simon (NSX) and Ben (vROPs). I hope I got their names right.

After we've settled in and got to know one another a little we discussed the vRA (vRealize Automations) product in depth and then did some labs to install the different components. The idea of the lab was to install all the windows components including certificates. One thing to note here: All the labs were based on the 6.x version of vRA even tough the latest version 7.0.1 was out for a couple of weeks. VMware changed a lot from 6.x to 7.0 especially in the installation part. So we had to struggle with the very complicated 6.x approach of installing everything and were told in some slides how very much less of a pain the same procedures were in the latest version.

Unfortunately due to the nested nature of our labs we've had a lot of issues with the installation. For me the installation failed completely and we didn't have time to troubleshoot the root cause. Five other guys from the course also destroyed their lab in a similiar fashion. The guys from LiveFire had a solution for that. They "timo-ed" our labs. Which was basically a re-deploy with a already configured vms.

**Day 2: vRealize Automations Configuration**

On the second day it was all about the inital configuration of vRA and some labs to implement that.

We've also talked about a distributed vRO (vRealize Orchestrator) installation and how to configure this. this brought us to the delta check vRA 6.x to 7.0. vRO is included in the 7.0 vRA appliance and doesn't have to be deployed separately anymore. There are still two use cases to do so: having a separate instance for each tenant or having them distributed geographically for concurrency and high availability reasons. There was a lab in which we configured the high availability of vRO.

**Day 3: NSX or "How to explain NSX to your grandmother"**

On the third day we've had a presentation on how to explain NSX to your grandmother. It was a very nice presentation which they gave us as well to explain it to our customers (or their grandmothers ^^).

After the presentation and some follow-ups by simon we did two labs on NSX. They were very basic, but all in all it was at least to me a very nice introduction into what NSX can do in a SDDC environment. (haven't had any hands-on experience at all).

**Day 4: vRealize Operations Manager, LogInsight and 10 minutes of Infrastructure Navigator**

On the fourth day the topic was vROps. Ben finally got to talk (by his own account he loves to hear his own voice) about the vRealize Operations Manager.

After a hour or two of slides Ben asked the class if he should show some advanced stuff in his own lab or if we should do the labs by ourselves. So up until lunch he showed us some very advanced things that you can do in vRealize Operations Manager and how simple it can be if you just keep in mind a couple of things.

After lunch we were left to do our labs. Just before the 3pm break Ben squeezed in 10 minutes about infrastructure navigator and what it can do for you. I never knew about the product and I was glad to get some information about it. basically it allows you to see relationships between your different VMs. this way you could probably discover some VMs that are not say SRM protected that talk a lot to VMs that are protected by SRM. So you'd need to figure out how important these VMs were to each other and maybe change the protection on some of them. Very interesting.

**Day 5: Advanced NSX and the Integration of vRA, NSX, vIN, vLI, vRO and vROps**

The last day was actually only half a day. Most people had to leave early to catch their flights. We've done some more advanced labs with NSX and micro segmentation. and in the end we created a blueprint which leverages the NSX integration to create a loadbalancer automatically for a requested vApp.

After that we had a big discussion around the feedback for the course which was overwhelmingly positive.

I really loved the course and learned a ton about the entire integration of all the SDDC products VMware has to offer. If you are reading this and thinking about wether or not to register for this course: Do it! Now! You won't regret it. Because you are also going to meet some very good people working in the same industry as you are. So you might make some friends to meet up with at the next VMworld!

Anyways, great Course. If you're interested in anything that I haven't really covered in this post feel free to hit me up in the comments.

[gallery ids="74,73,72,71,70,69,68,67,66,65" type="rectangular" orderby="rand"]
