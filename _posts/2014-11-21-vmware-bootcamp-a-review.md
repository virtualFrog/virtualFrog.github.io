---
layout: post
title: VMware Bootcamp. A Review
date: 2014-11-21 12:31:43.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Bootcamp
- VMware
tags: []
meta:
  _wpas_skip_facebook: '1'
  _wpas_skip_google_plus: '1'
  _wpas_skip_linkedin: '1'
  _wpas_skip_tumblr: '1'
  _wpas_skip_path: '1'
  _thumbnail_id: '6'
  geo_latitude: '47.567442'
  geo_longitude: '7.597551000000067'
  geo_address: Matthäus, Basel, Switzerland
  geo_public: ''
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  publicize_twitter_user: virtual_dd
  publicize_twitter_url: http://t.co/RfUkzPkcKx
  _wpas_done_9383931: '1'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:350746208;b:1;}}
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2014/11/21/vmware-bootcamp-a-review/"
---
Last week I was attenting my first VMware Bootcamp. The Bootcamp consisted of two courses: "VMware vSphere 5.5 - Fast Track" and "VMware vSphere 5.5 - Optimize and Scale".

First of all, let me address the terrible hotel-choice of the organizing company. We stayed at "Waldhotel Doldenhorn" in Kandersteg, Switzerland. For the first course there were 11 attendees, but the hotel forgot to reserve one of the meeting rooms for us so we were forced to cramp ourselved into a improvised meeting-room which usually would be a suite in the hotel<!--more--> (doesn't sound too bad, right? Well, that suite is designed for two people.. not 12 people all with their laptops and other course-material). So anyway, it was really nice and cozy..;) The food was very strange. The hotel tried to be "upper class" with its food, but all it accomplished was that we were not sure what they put on our plates. The service in the afternoon was also very poor.&nbsp; Everyday our teacher had to go and ask for water and coffee which was supposed to be brought without our teacher telling them to.

The teacher was excellent! His name is Urs Stephan Alder (http://www.kybernetika.ch/) and he really knew his way around the entire VMware Software Suite. His teaching skills are really good as well. In a class with 11 attendees there is bound to be a great gap in practical skills. I for example have been working in a production environment consisting of over 1500 VMs running on well over 130 ESXi Hosts while other in the course have never even seen the vSphere Client Console. Urs handled this very well and gave the advanced users some advanced labs while the "VMware-noobs" were being taught the basics.

At around half time of this 9-day bootcamp there was the first exam. Everyone in the class took the exam and passed it. Wonderful, we have 11 more VCP-DCV (VMware Certified Professional) now. I'll be doing a follow-up post about the exam in detail and maybe a few hints for people wanting to get their VCP-DCV soon.. ;) Stay tuned.

After that exam we were only 4 students left. We started with the course "VMware vSphere 5.5 - Optimize and Scale". The course is very interesting, especially if you have a big environment and want to increase your performance. Did you know, for example that there is a really big CPU-Overhead if a VM running a single-thread application is provisioned with two vCPUs? The overhead is so bad in fact, that VM will perform poorer on a dual-CPU than it would with only one vCPU!

Other interesting tidbits included Use-Cases for the tagging mechanism for VMs. We are currently undertaking a mini-project where we want to get all the VM-Owner Data to the VMs in form of tags. We'll use powershell for that and I will write a post about it once it is done and tested.

After some intensive days working on that course the four of us had to go to Berne, Switzerland to take our exam for VCAP-DCA (VMware Certified Advanced Professional - Data Center Administration). This exam was much different from the first. There were no Multiple Choice Questions! Only 23 Labs in a nested environment over at Palo Alto. The Labs were mixed together randomly. So in one lab we had to make some command-line-kung-fu to define custom satp rules while in one other lab we only had to define some very basic DRS Settings. (I unfortunately cannot disclose the exact phrasing of the lab exercises).

After only four hours I was notified by mail that I am now officially a VCAP - DCA.

Stay tuned for more updates on the exams and my daily-life as a VCAP.

