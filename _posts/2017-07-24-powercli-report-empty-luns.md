---
layout: post
title: 'PowerCLI: Report Empty LUNs'
date: 2017-07-24 13:19:08.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Scripting
- virtualFrog Posts
- VMware
tags:
- PowerCLI
meta:
  _thumbnail_id: '29'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:56:"https://twitter.com/virtual_dd/status/889444839561060352";}}
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '7428958144'
  _publicize_done_9340485: '1'
  _wpas_done_9383931: '1'
  publicize_twitter_user: virtual_dd
  publicize_linkedin_url: https://www.linkedin.com/updates?discuss=&scope=391645417&stype=M&topic=6295210531707973632&type=U&a=EZuM
  _publicize_done_14862392: '1'
  _wpas_done_14749148: '1'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2017/07/24/powercli-report-empty-luns/"
---
Once again I'm writing about a customer use-case. In this case the customer wanted to know if all of his many LUNs were actually being used or if there were any they could delete and save some storage space.

The customer has a very big environment so it would have been tedious to do this task by hand. PowerCLI comes to the rescue in form of this very short script that will basically just put together some information and send it to you by email.

<!--more-->

Update: this script is now published at [GitHub](https://github.com/virtualFrog/PowerCLI-Scripts).

[code language="powershell"]

##################################################################################  
# Script: Get-Empty-LUNs.ps1  
# Datum: 24.07.2017  
# Author: Bechtle Schweiz AG (c) 2017  
# Version: 1.1  
# History: Initial Script  
# Changed values to share it online  
##################################################################################

# vCenter Credentials koennen mit folgendem Command vorgaengig einmalig konfiguriert und hinterlegt werden  
# New-VICredentialStoreItem $vCenter  
# Default Parameter in erster "Param" Sektion anpassen, ansonnsten werden die hinterlegten Default Werte verwendet

[CmdletBinding(SupportsShouldProcess=$true)]  
Param(  
 [parameter()]  
 [Array]$vCenter = @("vCenter 1","vCenter 2"),  
 # Change to a SMTP server in your environment  
 [string]$SmtpHost = "mailserver",  
# Change to default email address you want emails to be coming from  
 [string]$From = "here\_is\_your\_from\_email\_address",  
# Change to default email address you would like to receive emails  
 [Array]$To = @("first","second"),  
# Change to default Report Filename you like  
 [string]$Attachment = "$env:temp\Empty-LUN-Report-"+(Get-Date –f "yyyy-MM-dd")+".csv"  
)

Import-Module -Name VMware.VimAutomation.Core -ErrorAction SilentlyContinue | Out-Null  
#There are multiple ways to achieve this. see other blog posts for more  
$cred = D:\sw\Script\PowerShell\VMWare\Get-myCredentials.ps1 domain\user D:\sw\Script\PowerShell\VMWare\cred\_doeda\_dmz

Connect-VIServer $vCenter -Credential $cred -WarningAction SilentlyContinue | Out-Null  
Write-Host "Connected to $vCenter. Starting script"

$bodyh = (Get-Date –f "yyyy-MM-dd HH:mm:ss") + " - the following Datastores were found to be empty. `n"  
$body = foreach ( $cluster in Get-Cluster) {Get-Datastore -RelatedObject $cluster |? {($\_ |Get-VM).Count -eq 0 -and $\_ -notlike "\*rest\*" -and $\_ -notlike "\*\_local" -and $\_ -notlike "\*snapshot\*" -and $\_ -notlike "\*placeholder\*"}|select Name, FreeSpaceGB, CapacityGB, @{N="NumVM";E={@($\_ |Get-VM).Count}}, @{N="LUN";E={($\_.ExtensionData.Info.Vmfs.Extent[0]).DiskName}}, @{N="Cluster";E={@($cluster.Name)}} |Sort Name }  
$body | Export-Csv "$Attachment" -NoTypeInformation -UseCulture  
$body = $bodyh + ($body | Out-String)  
$subject = "Report - Emtpy Datastores"  
send-mailmessage -from "$from" -to $to -subject "$subject" -body "$body" -Attachment "$Attachment" -smtpServer $SmtpHost  
Disconnect-VIServer -Server $vCenter -Force:$true -Confirm:$false  
[/code]

