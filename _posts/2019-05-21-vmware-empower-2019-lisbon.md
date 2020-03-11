---
layout: post
title: VMware EMPOWER 2019, Lisbon
date: 2019-05-21 13:42:53.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- vExpert
- virtualFrog Posts
- VMware
tags: []
meta:
  _wp_old_date: '2019-05-21'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '33305907288'
  timeline_notification: '1564168382'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:57:"https://twitter.com/virtual_dd/status/1132915236170850304";}}
  _publicize_done_9340485: '1'
  _wpas_done_9383931: '1'
  publicize_twitter_user: virtual_dd
  publicize_linkedin_url: ''
  _publicize_done_21519738: '1'
  _wpas_done_22130612: '1'
  _thumbnail_id: '1240'
  _wpas_skip_9383931: '1'
  _wpas_skip_22130612: '1'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2019/05/21/vmware-empower-2019-lisbon/"
---
In my third week working for my new employer I was sent to the VMware EMPOWER Partner Conference in Lisbon.

My new company sent me along with 3 technical colleagues and a couple of sales guys here which demonstrates the huge commitment to VMware.

## Day 0: Arriving in Portugal

We arrived a day early on Sunday and stayed at the Hotel closest to the venue. The hotel itself is quite nice but unfortunately lacks any facilities like a gym. After arriving on Sunday evening we started exploring around the hotel and found a cool spot: the LX factory.

This is a gated community of local pop up shops and restaurants with many delicious looking things and artsy stuff to buy. We ended up in a small restaurant to get a small bite. The place was decorated very nice and had some funny phrases in the menu:

> A meal without wine is breakfast

or

> Okay, I think this will make sense if I get more wine

We ended up getting some very delicious food and had a good time.

Afterwards we researched a sports bar to watch the Swiss national ice hockey team's match against Russia. Unfortunately the Swiss team lost 3:0 but the experience in the local sports bar was nevertheless very interesting.

After watching the match our group split up. While some guys went back to the hotel two colleagues and I started exploring the city some more and ended up getting a good-night snack at Burger King.

&nbsp;

## Day 1: The conference starts

On Monday the conference officially started. Very similar to VMworld conferences the attendees get access to a session builder beforehand and can choose their sessions based on their interests. My first session started at 1pm.

### So, you think you want to be a VCDX? What it really takes

This EMPOWER conference is the first VMware events with partner-led sessions. So due to my interest in the VCDX certification I signed up for the session with that title.

The session's speaker were 4 current VCDXs:

- Satvinder Lall, VCDX #157 (NV Track)
- Yves Sandfort, VCDX #203 (CMA Track)
- Marco van Baggum, VCDX #223 (DCV Track)
- Johan van Amersfoort, VCDX #238 (DTM Track)

As you can see, all four current tracks were there. The session itself was very interesting. Yves Sandfort moderated the session and talked about the VCDX program and how to get started. The others pitched in where they could and shared their experiences and gave some tips.

The main message to get started was: Read the blueprint!

If you don't read the blueprint you will have no idea what is required.

I will share some other notes that I took in that session in no particular order:

- VCDX is not a technical certification. It is a validation of your design and architecture skills
- You don't need to know the VMware products down to the advanced settings. A understanding of the technology is sufficient.
  - "Do I need to be able to create a vRealize Orchestrator workflow?"
  - "No, but you should be able to describe the architectural components so that someone else can build it"
- You need to understand the impact of your choices. For example: What is the impact on my design if I go with Storage vendor A compared to Storage vendor B?
- In the Defense there is no "I did not do this design" excuse. You need to know everything and be able to defend every part of the design
- A VCDX certification is basically a job guarantee.
- A design always starts with requirements and constraints
- Always talk to the business.
- Everything and everyone has an SLA. If someone says there is no SLA:
  - "Okay, let's shut down your datacenter and leave it down for 6 months. That is no problem, right?"
- In the future there will be benefits for VMware partners with VCDXs on staff. (Currently there is no benefit)

That's it. All in all a very interesting and fun session which ignited a new spark of motivation to pursue this certification in the near future. (I had started working on it and stopped because my design was not suitable at the time.)

### VCDX War Stories and Open Q&A

This session took place right after the previous one with the same set of VCDX speakers. This one had a different feel to it because the speakers presented some myths around the VCDXs and shared their experiences on some topics both before and after they achieved their VCDX.

In this session I did not have as many things to take away as in the first session but nevertheless here are my notes:

The most challenging design or scenario that they have faced:

- Getting Cisco ACI and VMware NSX to play nice together
- Physically impossible tasks (vMotioning VMs with a greater RAM Change rate than the available network bandwidth)
- 100% Uptime

They mentioned that the more complex your design is the more likely you are to fail in the defense. This makes sense if you think about it. With more complexity you open yourself up to more "what if" questions. So one advice was: Keep your design as simple as possible (always fulfilling the requirements).

At some point in this session I asked a question:

"At what point in the process should one seek out a mentor?"

They answered differently. Yves said when the design document is around 60,70,80% done. And Johan said that he waited too long and said that you should seek one as soon as possible.

### VCAP-DCV Exam Tips

This session was not very good. I am going to take the VCAP-DCV 6.5 Deploy Exam on Thursday so I thought some tips could come in handy. In this session however they were (of course) not allowed to talk about the contents of the exam but focused on some generic tips around getting to know the HOL interface and be prepared to find stuff in the provided PDFs.

Here is a list of my notes (I stopped taking notes after a while):

1. 
  1. Documentation is available as PDFs
    1. There are around 20 different PDFs

Be familiar with the HOL (Hands-on Labs) interface

Get to know all the "Hacks" to maximize the screen real-estate
The best study is trial and error (retake the exam if you fail)
Make use of the provided notepad and marker
1. 
Use a checklist to track time 1. 
    1. There are 17 items in the exam: 205 minutes - 12 minutes per item

The time runs out very fast

Complete the items you are familiar with first
Try not to break the lab
Some items are prerequisites to other items
Read the blueprint!
Report it if the environment hangs
Review your work if the time allows for it
### General Session Day 1

At the end of the first day was the general session. The session started out really bad with a speaker from VMware with a heavy french accent. Very hard to get used to and understand.

The session was very long and they talked about the launch of a new partner program in 2020 and all the benefits the partners will get from that.

Afterwards another guy from VMware talked about the innovations from the past decades in VMware technology and how innovations in one layer will reduce the complexity of innovating at the next layer.

I have to admit that I lost track of all the things that were said in the GS. We left the General session after about an hour and it went on (without us) for another half hour or so.

## Day 2: More sessions and a mention in the General Session

On the second day my schedule was pretty easy. My first session started 11:45 AM so I had plenty of time for breakfast and squeezing in some prep time for the VCAP-DCV Deploy exam that I will take on Thursday.

### SDDC for Cloud Management

My first session on day two. Unfortunately the speakers were marketing people. They did not really know the technical side of the VMware Cloud Foundation (someone in the audience had to tell them that the product has an integration into HPE OneView). I did not see much value in getting marketing knowledge on this topic so I left the session early and went to work on this blog post.

### Portuguese Lunch

We got together with a colleague from another company in switzerland and went outside the venue for lunch. We ended up in a Portuguese restaurant and had a very good lunch. However the mentality of the staff there was very relaxed so we all ended up missing our session after lunch. I was scheduled for "Introduction to PKS" but I guess I can get that knowledge in the **BUSINESS IT** Multi-Cloud Experience Platform (MEP).

### Cloud Automation - The Road to Multi-Cloud

I was looking forward to this session the most on day two. It talks about the Cloud automation platform that VMware is heavily working on and ties very well into the Multi-Cloud Experience Platform (MEP) that we are building at **BUSINESS IT**.

[EXPAND THIS TOPIC MORE]

### General Session Day 2

The second general session took place at the end of the day. To be honest I did not follow everything that was said (I was working on this blog post on my tablet) but then an incredible thing happened. The vice president of some part of EMEA mentioned **BUSINESS IT** from switzerland as the top partner with whom they realized big projects with IBM. They basically said that with the help of **BUSINESS IT** they were able to deliver expert knowledge in a huge project for IBM where **BUSINESS IT** worked in the name of VMware.

We actually do a lot of work for VMware as part of the PSO (Professional Services Organization). And we get these PSO engagements thanks to our commitment to VMware and the achievement of **3** of 4 of the **Master Services Competencies**.

### EMPOWER Party at LX Village

After the General Session it was time for the celebration. VMware has organized a very cool venue to host the 1700ish partners that came to portugal. There was a nice variety of snacks and some retro games to have a friendly competition at. We connected with some guys from VMware switzerland and some colleagues from other partner firms and had a really good time.

Time ran fast and before long the venue was closing and everybody had to leave at around 11pm. We were very close to our hotel so we walked back and ended up at the hotel bar which soon thereafter closed as well. Before we headed back to our hotel rooms we met some charming ladies from VMware which work at the exam center here in lisbon. We had a cool chat and ended up promising to each take an exam on-site at EMPOWER to bump their numbers.

I was back in my room at around half past midnight.

## Day 3: A tightly packed schedule and some hands-on time

The third day started comparatively early for me. My fist session started at 08:30 AM and I had to squeeze in some breakfast before that. One of my coworkers had booked the same session so we headed over to the venue together.

### VCDX Session: Best Practices in Multi-Cloud Automation

The first session was led by a VCDX [ENTER VCDX INFO HERE] who started the session by saying "Cool that more than 1 person is attending, makes me feel much better about presenting after the party last night". I think the room about a third full.

He went on to talk about the VCDX way to talk to a customer.

[MORE DETAILS ON THE SESSION]

### Automating NSX-T and PKS for N00bs

This was the title of our second session that day. It turned out however than none of us (3 from BUSINESS IT attended this session) read the description of the session. The session was all about a tool called "concourse" which lets you automatically deploy NSX-T and PKS components while being a container application itself.

Because none of us knew the product we did not really get anything out of this session.

### Taking the VCAP-DCV 6.5 Design Exam and landing a 300

After the "N00bs" session we went to make good on our promise from the night before and headed for the exam center. I ended up signing up for the "VMware Advanced Professional Data Center Virtualization 6.5 Design Exam" and taking it right then without any sort of preparation. We originally planned to take the deploy exam while in lisbon but due to the lab based nature of the exam we would have had to travel to one of lisbon's certified exam centers (the closest was about 30 minutes away from the EMPOWER venue). The test was changed since the last time I took it and removed the rudimentary "visio" tool that many people seemed to struggle with. The test is now pretty much all multiple choice.

After answering all questions in about 40 minutes I had a small list of questions which I flagged for review during the test. After re-reading the first one and still not knowing a better answer I decided to just finish the test.

My score was a "perfect" 300. For those that do not know: 300 is the minimum required amount of points to pass. I would have failed with 299 points but still passed with 300 points!

So I was pretty happy about passing this exam even if it was a very close call.

### vSAN 6.7 Challenge Lab

The next session on my schedule was a vSAN challenge lab. Everybody needed to bring their own laptop and go to https://labs.hol.vmware.com and there look for the challenge lab 1908 for vSAN.

The speaker of the session really knew his vSAN and was telling everyone lots of details around vSAN and which configurations made sense and which didn't.

I did not really learn much but it was a cool atmosphere in there.

### HCI Assessment (LiveOptics) Technical Deep Dive

The next and last session of the day was all about the HCI assessment tool Live Optics.

[CONTENT FROM SESSION]

