---
layout: post
title: 'PowerCLI: Find differences between DNS-Name and VM Display-Name'
date: 2014-12-09 14:05:04.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Scripting
- VMware
tags:
- PowerCLI
meta:
  _wpas_skip_facebook: '1'
  _wpas_skip_google_plus: '1'
  _wpas_skip_linkedin: '1'
  _wpas_skip_tumblr: '1'
  _wpas_skip_path: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  publicize_twitter_user: virtual_dd
  publicize_twitter_url: http://t.co/QSKfrgosff
  _wpas_done_9383931: '1'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:350746208;b:1;}}
  _thumbnail_id: '29'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2014/12/09/powercli-find-differences-between-dns-name-and-vm-display-name/"
---
<p>My boss asked me to create a report which shows all the VMs that have a difference in DNS-hostname and the displayname shown in the client. The reason is simple: our backup solution sometimes has troubles when those values differ. And in some rare occassion we cannot find the VM on which we should be doing support on because the client only knows the dns-name and we can only find VMs by displayname.<br />
<!--more-->Update: this script is now published at <a href="https://github.com/virtualFrog/PowerCLI-Scripts" target="_blank" rel="noopener">GitHub</a>.</p>
<p>Long story, short script:</p>
<p>[code language="powershell"]<br />
$OutArray = @()</p>
<p>$vms = Get-VM<br />
foreach ($vm in $vms)<br />
{<br />
$myobj = "" | Select "VM", "DNSname", "Status"<br />
$guest = Get-VMGuest -VM $vm<br />
$pat = "."<br />
$vmname = $guest.VmName<br />
$hostname = $guest.HostName<br />
$myobj.VM = $vmname<br />
$myobj.DNSname = $hostname<br />
if ( $hostname -ne $null)<br />
{<br />
$pos = $hostname.IndexOf(".")<br />
if ( $pos -ne "-1")<br />
{<br />
$hostname = $hostname.Substring(0, $pos)<br />
}<br />
if ( $hostname -ne $vmname )<br />
{<br />
Write-Host -ForegroundColor Red "---&gt; The VM: $vm has different DNS and Display-Name!"<br />
Write-Host -ForegroundColor Red "----&gt;Hostname: " $hostname<br />
Write-Host -ForegroundColor Red "----&gt;VmName: "$vmname<br />
$myobj.Status = "Not OK!"<br />
}<br />
Else<br />
{<br />
Write-Host -ForegroundColor Green "---&gt; The VM: $vm has identical Names."<br />
$myobj.Status = "OK!"<br />
}<br />
}<br />
Else<br />
{<br />
Write-Host -ForegroundColor Yellow "---
\> The VM: $vm is not powered-on. Hostname cannot be found in this state!"  
$myobj.Status = "N/A"  
}  
Clear-variable -Name hostname  
Clear-variable -Name vmname  
$OutArray += $myobj  
}  
$OutArray | Export-Csv "c:\tmp\name\_vs\_dns\_$vcenter.csv"  
[/code]  
the script connects to a vcenter server of your choice (fill the $vcenter variable!) and then reads out all the vms. it then goes through each vm and gets the guest object of each VM. after that we simply put together our values into variables.  
since dns names always differ from hostnames when dns is fqdn, I go ahead and count the dots (.) in the dns-name. if there are some, they will be cut away, otherwise it's fine. if the dns-hostname is $null it means that the VM is powered off or doesn't have the necessary tools installed.  
this is one of my first powershell scripts, so I guess I could have made it work with a few less variables, but hey, it works.. :)

