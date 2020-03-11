---
layout: post
title: 'PowerCLI: Automating VM Shutdown triggered by USV'
date: 2017-07-21 10:00:00.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- virtualFrog Posts
tags: []
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _thumbnail_id: '29'
  _publicize_done_9340485: '1'
  _wpas_done_9383931: '1'
  _publicize_job_id: '7321226599'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:56:"https://twitter.com/virtual_dd/status/888311971719852032";}}
  publicize_twitter_user: virtual_dd
  publicize_linkedin_url: https://www.linkedin.com/updates?discuss=&scope=391645417&stype=M&topic=6294077662973435904&type=U&a=QaoA
  _publicize_done_14862392: '1'
  _wpas_done_14749148: '1'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2017/07/21/powercli-automating-vm-shutdown-triggered-by-usv/"
---
So another customer had a requirement. They want their USV to trigger a script that moves all VMs from one datacenter to the other datacenter in case of a power failure.

I started to write the following script, but it has not been finalized because some of the customer's engineers are on holidays. I will update this post as soon as the final script is ready.

One thing to note: It uses the same VI Credential Store as my previous script to automate VM skeleton creation.

<!--more-->

Update: this script is now published at [GitHub](https://github.com/virtualFrog/PowerCLI-Scripts).

[code language="powershell"]

############################################################################################  
# Script name: EvacuateVMsFromUSV.ps1  
# Description: Evacuate all VMs from one site in a active-active cluster and shut down the Hosts  
# Version: 1.0  
# Date: 20.07.2017  
# Author: Bechtle Steffen Schweiz AG | Dario Doerflinger (virtualfrog.wordpress.com)  
# History: 20.07.2017 - First tested release  
############################################################################################

# Example: # e.g.: .\EvacuateVMsFromUSV.ps1 -SiteToShutdown Allschwil

param (  
 [string]$SiteToShutdown # Identifier of site  
)  
$vCenter\_server = "bezhvcs03.bechtlezh.ch"  
# clear global ERROR variable  
$Error.Clear()

# import vmware related modules, get the credentials and connect to the vCenter server  
Import-Module -Name VMware.VimAutomation.Core -ErrorAction SilentlyContinue | Out-Null  
Import-Module -Name VMware.VimAutomation.Vds -ErrorAction SilentlyContinue |Out-Null  
$creds = Get-VICredentialStoreItem -file "C:\Users\Administrator\Desktop\login.creds"  
Connect-VIServer -Server $vCenter\_server -User $creds.User -Password $creds.Password |Out-Null

# define global variables

$current\_date = $(Get-Date -format "dd.MM.yyyy HH:mm:ss")  
$log\_file = "C:\Users\Administrator\Desktop\\log\_$(Get-Date -format "yyyyMMdd").txt"

Function SetDRStoAutomatic ($cluster)  
{  
 try {  
 $cluster | Set-Cluster -DrsEnabled:$true -DrsAutomationLevel FullyAutomated -Confirm:$false |Out-Null  
 } catch {  
 Write-Host -Foregroundcolor:red "Could not set DRS Mode to automatic"  
 }  
}

Function RemoveRemovableMediaFromVMs($esxhost)  
{  
 try {  
 $esxhost | Get-VM | Where-Object {$\_.PowerState –eq “PoweredOn”} | Get-CDDrive | Set-CDDrive -NoMedia -Confirm:$False |Out-Null

} catch {  
 Write-Host -Foregroundcolor:red "Could not get the vm objects from host."  
 }  
}

Function EvacuateVMsFromHost($esxhost)  
{  
 try {  
 $esxhost | Set-VMHost -State Maintenance -Evacuate:$true -Confirm:$false |Out-Null  
 } catch {  
 Write-Host -Foregroundcolor:red "Could not put host into maintenance mode"  
 }  
}

Function ShutDownHost($esxhost)  
{  
 try {  
 $esxhost | Stop-VMhost -Confirm:$false -Whatif  
 } catch {  
 Write-Host -Foregroundcolor:red "Could not shut down host"  
 }  
}

###### Main Program ######  
if ($SiteToShutdown -eq "Allschwil") {  
 $hosts = @("bezhesx40.bechtlezh.ch")

} elseif ($SiteToShutdown -eq "Pratteln")  
{  
 $hosts = @("bezhesx41.bechtlezh.ch")  
}

foreach ($esxhost in $hosts)  
{  
 $cluster = (get-vmhost $esxhost).Parent  
 SetDRStoAutomatic($cluster)

$esxihost = Get-VMhost $esxhost  
 RemoveRemovableMediaFromVMs($esxihost)  
 EvacuateVMsFromHost($esxihost)  
 ShutDownHost($esxihost)  
}

# cleanup and removal of loaded VMware modules  
Disconnect-VIServer -Server $vCenter\_server -Confirm:$false |Out-Null  
Remove-Module -Name VMware.VimAutomation.Vds -ErrorAction SilentlyContinue | Out-Null  
Remove-Module -Name VMware.VimAutomation.Core -ErrorAction SilentlyContinue | Out-Null

# write all error messages to the log file  
Add-Content -Path $log\_file -Value $Error

[/code]

