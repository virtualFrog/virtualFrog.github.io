---
author: virtualfrog
comments: true
date: 2016-06-16 12:28:07+00:00
layout: post
link: https://virtualfrog.wordpress.com/2016/06/16/esxi-kickstart-new-ways-to-set-vmotion-fault-tolerance-logging-etc/
slug: esxi-kickstart-new-ways-to-set-vmotion-fault-tolerance-logging-etc
title: 'ESXi KickStart: New Ways to set vMotion, fault Tolerance Logging, etc.'
wordpress_id: 35
categories:
- Kickstart
- VMware
tags:
- xSA
---

Hello guys!

![scripting-platforms](https://virtualfrog.files.wordpress.com/2016/06/scripting-platforms.jpg)

I'm back on my blog after changing my job and generally having been very busy over the last two years.<!-- more -->

Today I want to tell you guys about change in esxcli (which happened sometime in the last two years) which allows you to set vMotion, Management and for example fault tolerance Logging on a vmkernel interface in VMware ESXi.

In the past you had to do this with vim-cmd for fault tolerance and vmotion and had to use some other tricks to get this working with the mangement traffic.

Now however this is a simple esxcli command.

Here are the functions for vmotion that I've (re-)written for my esxi staging appliance.


<blockquote># Configuring vMotion
SetVMotion()
{
# Function variables, if not defined uses standard values
portGroupName=$1
if [ ! -z $2 ]; then mtu=$2; else mtu=1500; fi
vmkIPs=$myVMIP1,$myVMIP2
# For every port group with that name, a vmk will be generated and configured for VMotion
i=1
for portGroup in `esxcli --formatter=csv network vswitch standard portgroup list | grep -i $portGroupName | awk -F "," '{print $2}'`; do
esxcli network ip interface add -p $portGroup -m $mtu
case $esxVersion in
"6.0.0"|"5.5.0")
vmk=`esxcli --formatter=csv network ip interface list | grep -i $portGroup | awk -F "," '{print $5}'`
;;
"5.0.0"|"4.1.0")
vmk=`esxcli --formatter=csv network ip interface list | grep -i $portGroup | awk -F "," '{print $4}'`
;;
*);;
esac
vmkIP=`echo $vmkIPs | awk -F "," '{print $'$i'}'`
esxcli network ip interface ipv4 set -i $vmk -I $vmkIP -N $myVMMask -t static
**esxcli network ip interface tag add -i $vmk -t VMotion**
i=$(( $i + 1 ))
done
}</blockquote>


The command in bold can be found in the latest documentation:

[Here](http://pubs.vmware.com/vsphere-60/index.jsp#com.vmware.vcli.ref.doc/esxcli_network.html)
