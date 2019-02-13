---
author: virtualfrog
comments: true
date: 2014-12-09 12:05:04+00:00
layout: post
link: https://virtualfrog.wordpress.com/2014/12/09/powercli-find-differences-between-dns-name-and-vm-display-name/
slug: powercli-find-differences-between-dns-name-and-vm-display-name
title: 'PowerCLI: Find differences between DNS-Name and VM Display-Name'
wordpress_id: 24
categories:
- Scripting
- VMware
tags:
- PowerCLI
---

My boss asked me to create a report which shows all the VMs that have a difference in DNS-hostname and the displayname shown in the client. The reason is simple: our backup solution sometimes has troubles when those values differ. And in some rare occassion we cannot find the VM on which we should be doing support on because the client only knows the dns-name and we can only find VMs by displayname.
<!-- more -->Update: this script is now published at [GitHub](https://github.com/virtualFrog/PowerCLI-Scripts).

Long story, short script:

[code language="powershell"]
$OutArray = @()

$vms = Get-VM
foreach ($vm in $vms)
{
$myobj = "" | Select "VM", "DNSname", "Status"
$guest = Get-VMGuest -VM $vm
$pat = "."
$vmname = $guest.VmName
$hostname = $guest.HostName
$myobj.VM = $vmname
$myobj.DNSname = $hostname
if ( $hostname -ne $null)
{
$pos = $hostname.IndexOf(".")
if ( $pos -ne "-1")
{
$hostname = $hostname.Substring(0, $pos)
}
if ( $hostname -ne $vmname )
{
Write-Host -ForegroundColor Red "---> The VM: $vm has different DNS and Display-Name!"
Write-Host -ForegroundColor Red "---->Hostname: " $hostname
Write-Host -ForegroundColor Red "---->VmName: "$vmname
$myobj.Status = "Not OK!"
}
Else
{
Write-Host -ForegroundColor Green "---> The VM: $vm has identical Names."
$myobj.Status = "OK!"
}
}
Else
{
Write-Host -ForegroundColor Yellow "---> The VM: $vm is not powered-on. Hostname cannot be found in this state!"
$myobj.Status = "N/A"
}
Clear-variable -Name hostname
Clear-variable -Name vmname
$OutArray += $myobj
}
$OutArray | Export-Csv "c:\tmp\name_vs_dns_$vcenter.csv"
[/code]
the script connects to a vcenter server of your choice (fill the $vcenter variable!) and then reads out all the vms. it then goes through each vm and gets the guest object of each VM. after that we simply put together our values into variables.
since dns names always differ from hostnames when dns is fqdn, I go ahead and count the dots (.) in the dns-name. if there are some, they will be cut away, otherwise it's fine. if the dns-hostname is $null it means that the VM is powered off or doesn't have the necessary tools installed.
this is one of my first powershell scripts, so I guess I could have made it work with a few less variables, but hey, it works.. :)
