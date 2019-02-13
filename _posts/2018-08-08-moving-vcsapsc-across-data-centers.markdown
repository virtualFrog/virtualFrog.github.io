---
author: virtualfrog
comments: true
date: 2018-08-08 15:49:49+00:00
layout: post
link: https://virtualfrog.wordpress.com/2018/08/08/moving-vcsapsc-across-data-centers/
slug: moving-vcsapsc-across-data-centers
title: Moving VCSA&PSC across data centers
wordpress_id: 781
categories:
- General
- VMware
---

Today I was at one of our customers that use Zerto as their BC/DR software. The customer is in the process of migrating his workloads into two new data center locations. The old environment was a stretched cluster and the new environment will be a stretched cluster as well. So from a Zerto perspective all VMs are replicated to "self".

Today the customer wanted to move the vCenter Server Appliance (VCSA) and the Platform Services Controller (PSC) VMs to the new environment. <!-- more -->Both VMs were already in a temporary cluster that was in the old data center but part of the new vCenter inventory. So we basically had to move the VMs from Cluster A to Cluster B within the vCenter inventory. The two clusters did not share storage with each other and also had different CPU types on their hosts.

To add a little difficulty to the whole process we also needed to change the IP address of the moved VMs because the new data centers had their own IP ranges.

The problem here is that Zerto depends on the vCenter, especially in a stretched cluster environment where only one vCenter server is present the vCenter is very critical. So we could not use zerto to fail over the vCenter server to the new site (the rest of the VMs are migrated using Zerto, which is a superb use-case for zerto by the way).

So after a little brainstorming we came up with the following plan:



	
  * copy/clone the VMs to the new site

	
  * assign a "internal" network to both cloned VMs

	
  * power on cloned VMs

	
  * change both VMs IP address over the vSphere console (I followed [this post](https://digitalkungfu.wordpress.com/2015/02/25/how-to-fix-vcsa-ip-settings-from-command-line/))

	
  * shut down cloned VMs

	
  * shut down source VMs (loose connection to vCenter obviously)

	
  * change cloned VMs network assignment to the new network (dvSwitch portgroup)

	
  * change DNS to reflect the VMs new IP address (very important!)

	
  * power on cloned VMs

	
  * sit back and enjoy the accomplished migration of those critical VMs


As with all plans, they never stick. First problem was that after we cloned the vCenter and PSC VMs we realized we needed to assign uplinks to the new network that the cloned VMs will use. So we did that knowing that it would be a change in the vcenter database that will be lost once we did a failover.
We ended up taking one uplink from one of the hosts in the new data center away from the dvSwitch and created a temporary standard switch with the needed portgroup. This would solve the problem of having no connectivity after booting up the VMs in the new data centers.

We then went ahead with the plan with those adjustments in place. The VMs were instantly replying to ping with their new IP addresses. That meant DNS and network configuration were working as planned. But here comes the next problem. The web client did not start. Usually when you reboot a VCSA you get some error messages until the web client is fully loaded, in this case we got a "cannot display web page" response.

After some digging around we found out that a lot of services did not come up. the "service-control --status" command from the appliancesh of the VCSA showed the following output:


<blockquote>root@VCSA [ ~ ]# service-control --status
Running:
lwsmd vmafdd
Stopped:
applmgmt vmcam vmonapi vmware-cm vmware-content-library vmware-eam vmware-imagebuilder vmware-mbcs vmware-netdumper vmware-perfcharts</blockquote>


So this showed us that the process of starting services was interrupted and could not continue. The command "service-control --start --all" provided us with the following error (truncated):


<blockquote>"localized": "An error occurred while starting service 'vmware-vmon'",
"translatable": "An error occurred while starting service '%(0)s'"
"Stderr: Failed to execute operation: Unit file is masked\n"</blockquote>


After doing some research on google we found a very helpful discussion in the VMware communities [here](https://communities.vmware.com/thread/563551). This proposed to "unmask" the vmon service and reboot the machine to solve the problem.

We ended up issuing the following command from the VCSA bash shell:


<blockquote>systemctl unmask vmware-vmon.service</blockquote>


This gave us back the message that it has removed the syslink to the service. After doing a reboot everything was fine.

Worth mentioning ist that the affected vCenter was running on the latest vSphere 6.5 build. (Update 2b, Build number 8815520).

This is a good example of how helpful it is to share knowledge. If no one had shared the tidbit about unmasking the service in order to mitigate the problem I might have stumbled upon it myself after some hours of researching the issue, or I might not have and could even have ended up re-installing the vCenter from scratch wasting a lot of time in the process. Sharing is caring.
