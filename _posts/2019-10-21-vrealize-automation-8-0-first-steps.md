---
layout: post
title: "[vRealize Automation 8.0] First Steps"
date: 2019-10-21 08:02:10.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- vExpert
- VMware
- vRealize Automation
tags:
- Cloud Assembly
- Firstlook
- vRA
meta:
  publicize_linkedin_url: ''
  _rest_api_client_id: "-1"
  _rest_api_published: '1'
  _thumbnail_id: '1294'
  _publicize_done_21519738: '1'
  _wpas_done_22130612: '1'
  _wpas_done_9383931: '1'
  publicize_twitter_user: virtual_dd
  _publicize_done_9340485: '1'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:57:"https://twitter.com/virtual_dd/status/1186163024400400384";}}
  timeline_notification: '1571638277'
  _publicize_job_id: '36565507696'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2019/10/21/vrealize-automation-8-0-first-steps/"
excerpt: I learn how the newly released vRealize Automation 8.0 works. I am documenting
  what I learn as I go by doing a Series of blog posts about this. Stay tuned.
---
Last Friday VMware released vRealize Automation 8.0. I have been looking forward to this release for quite some time because it was said that VMware will bring major changes to their Automation Engine. This week I have been fortunate to attend the Livefire Training for vRealize Orchestrator where I learned a ton of new things.

First off I want to clarify that I have installed this new version in our MEP (Multi-Cloud Experience Platform) and it is not intended for production. It is purely for educational purposes.

Let’s start off with the installation.

**Installation of vRealize Automation 8.0**

The first thing that has changed is that vRA 8.0 now utilises the Lifecycle Manager for the deployment of vRA. Another new thing is that it also requires the Identity Manager as well. So two new things to get your vRA goodness up and running.

The download of the package consists of an ISO image of almost 10 gb. After downloading the image the folder structure within the ISO basically resembles the one from a VCSA image. You have your installer executable for windows, Mac and linux platforms as well as an CLI installer. For this first installation I chose to use the UI instead of the CLI installer.

![vRA Installation Screen 1]({{ site.baseurl }}/assets/images/vra-installation-screen-1.png "vRA Installation Screen 1.png")

![vRA Installation Screen 2]({{ site.baseurl }}/assets/images/vra-installation-screen-2.png "vRA Installation Screen 2.png")

In the second Image above you can see what I have already described.

Going through each of the screens you will need to provide the vCenter along with credentials and FQDNs, IPs and the corresponding network information for each of these three appliances along with a password that will be used on all instances.

After all the information has been put into the assistant it will start the deployment. First up is the Lifecycle manager. After that comes the identity manager and lastly the vRA appliance. After everything is done you will have three different URL for those three appliances.

Next up: Configuring everything for the first time use.

**Initial configuration (or:&nbsp;“how can I&nbsp;login and start playing with this thing?”)**

The first thing I did was try and login to vRA. Https://\<your-vRA-fqdn\>

I have used the user “admin” and the password that I had supplied during installation. The login was a success but then a message told me that “You are not entitled to use cloud services”. All I could do at this point was log back out or reach out to support. I chose to log out.

I thought “Okay, maybe I need to check the other appliances first”. I went ahead and logged in to the Lifecycle Manager instance. This is actually the first time that I came in touch with this Appliance so I just clicked around and looked at all the available options. I could not find anything useful for my login problem so I logged back out.

Next I logged into the Identity Manager. This is where things started to make sense. I went ahead and configured our Active Directory domain for our MEP environment. The domain join went through fine. Then it wanted to know which groups or users to sync. Because this interface wanted the “DN” (Distinguished Name) of either the groups or the users I was lost for a minute because I had no idea how to get that information. Google to the rescue.

I stumbled upon an entry from IBM that basically told me to do this on my domain joint windows desktop:

- Run “lusrmgr.msc” to get to the local users and groups mmc
- Double click any of the groups in “groups” and try to add a new user
- Click advanced which gives you an advanced search window
- With a right click in the bottom half you can select which columns should be shown and voila there is a field called “Distinguished Name"

After that is selected I was able to search for users and groups and was presented with the full DN string for each of them.

Back to the Identity Manager interface and typing in the DN got me a step further. All my users have been imported.

Here is a crucial information that I actually gotten much later but I’ll not go into that. Under the now synced users for my active directory domain you can also see which “system domain” users have been created. One of them is the “configadmin@vmware.com” User that is needed to finalise the set up of the vRealize Automation 8.0 appliance.

So going back to the vRA Login page I now knew which user I needed to use in order to get a step further. In my case the user was named after my identity manger instance: ddo-idm. The password was the one I provided during setup.

Here I was: Logged in to the new vRealize Automation 8.0 UI.

![vRA UI Screen 1]({{ site.baseurl }}/assets/images/vra-ui-screen-1.png "vRA UI Screen 1.png")

**Quickstart Wizard**

The new UI can be a bit overwhelming because it looks very different than the old Automation UI. For starters we now have “Cloud Assembly”, “Service Broker” and other Services to choose from.

After first logging in you have the option to do the Quickstart Wizard which is recommended and takes just a few minutes to complete.

It basically configures your first on-prem vCenter endpoint and creates your first basic blueprint for you. This is great because you can learn quite a bit from those created elements later on.

In my setup I chose to skip the NSX endpoint and used a static dvSwitch network. I then chose to use my pre-created CentOS Template along with a pre-created customisation specification. One thing that vRA 8 still does not do is list the available customisation specifications. You still have to provide the exact name like before.

After filling in the information it goes ahead and deploys your first blueprint. Luckily everything went through smoothly and I had my first vRA deployed VM just minutes after finally getting my first login.

One thing that really stands out for me is this feature:

![vRA UI Screen 2]({{ site.baseurl }}/assets/images/vra-ui-screen-2.png "vRA UI Screen 2.png")

This depicts how your project will name the machines that are created within. This is purely awesome if you ask me. One thing we want to do in our MEP is that whenever someone requests a catalogue item the name of the VM(s) that are created should start with the username of the requester. This feature makes this easy as pie.

**Learning the first few things**

Okay, I got my first blueprint spoon-fed to me by the Quickstart wizard. Time to take matter into my own hands. I have a prepared Windows 2019 template that I would like to put into a blueprint. So I go ahead, create a new blueprint and then add a vSphere virtual machine to the canvas. What I could not figure out at first was: How do I now tell my blueprint which Template this should use and what customisation spec and so forth. All the thing the QuickStart wizard did for me. This is where the blueprint created from that QuickStart wizard starts to add value. I was able to go into this blueprint and learn how that blueprint has determined which template and which custom spec to use. This is now done by utilising properties:

![vRA UI Screen 3]({{ site.baseurl }}/assets/images/vra-ui-screen-3.png "vRA UI Screen 3.png")

I think I will need to invest some time into understanding how these properties are made up. This is essentially blueprint-as-code.

This is the end of my first steps into vRA 8.0 on the day it was released. I am very excited about learning more about this product and getting to know how I can use this in a Greenfield environment for our customers. Luckily a lot of our customers have not yet adopted much automation and a easier to use automation framework will go a long way in helping us convince them of their benefit if and when they decide to automate their infrastructure.

This was the first post in many posts to come as I learn about vRealize Automation 8.x.

