---
author: virtualfrog
comments: true
date: 2017-07-24 11:19:08+00:00
layout: post
link: https://virtualfrog.wordpress.com/2017/07/24/powercli-report-empty-luns/
slug: powercli-report-empty-luns
title: 'PowerCLI: Report Empty LUNs'
wordpress_id: 289
categories:
- General
- Scripting
- VMware
tags:
- PowerCLI
---

Once again I'm writing about a customer use-case. In this case the customer wanted to know if all of his many LUNs were actually being used or if there were any they could delete and save some storage space.

The customer has a very big environment so it would have been tedious to do this task by hand. PowerCLI comes to the rescue in form of this very short script that will basically just put together some information and send it to you by email.

<!-- more -->

Update: this script is now published at [GitHub](https://github.com/virtualFrog/PowerCLI-Scripts).

[code language="powershell"]

##################################################################################
# Script:           Get-Empty-LUNs.ps1
# Datum:            24.07.2017
# Author:           Bechtle Schweiz AG (c) 2017
# Version:          1.1
# History:          Initial Script
#                   Changed values to share it online
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
    [string]$From = "here_is_your_from_email_address",
# Change to default email address you would like to receive emails
    [Array]$To = @("first","second"),
# Change to default Report Filename you like
    [string]$Attachment = "$env:temp\Empty-LUN-Report-"+(Get-Date –f "yyyy-MM-dd")+".csv"
)

Import-Module -Name VMware.VimAutomation.Core -ErrorAction SilentlyContinue | Out-Null
#There are multiple ways to achieve this. see other blog posts for more
$cred = D:\sw\Script\PowerShell\VMWare\Get-myCredentials.ps1 domain\user D:\sw\Script\PowerShell\VMWare\cred_doeda_dmz

Connect-VIServer $vCenter -Credential $cred -WarningAction SilentlyContinue | Out-Null
Write-Host "Connected to $vCenter. Starting script"

$bodyh = (Get-Date –f "yyyy-MM-dd HH:mm:ss") + "  -  the following Datastores were found to be empty. `n"
$body = foreach ( $cluster in Get-Cluster) {Get-Datastore -RelatedObject $cluster |? {($_ |Get-VM).Count -eq 0 -and $_ -notlike "*rest*" -and $_ -notlike "*_local" -and $_ -notlike "*snapshot*" -and $_ -notlike "*placeholder*"}|select Name, FreeSpaceGB, CapacityGB, @{N="NumVM";E={@($_ |Get-VM).Count}}, @{N="LUN";E={($_.ExtensionData.Info.Vmfs.Extent[0]).DiskName}}, @{N="Cluster";E={@($cluster.Name)}} |Sort Name }
$body | Export-Csv "$Attachment" -NoTypeInformation -UseCulture
$body = $bodyh + ($body | Out-String)
$subject = "Report - Emtpy Datastores"
send-mailmessage -from "$from" -to $to -subject "$subject" -body "$body" -Attachment "$Attachment" -smtpServer $SmtpHost
Disconnect-VIServer -Server $vCenter -Force:$true -Confirm:$false
[/code]
