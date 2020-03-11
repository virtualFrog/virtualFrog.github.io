---
layout: post
title: 'PowerCLI: Adding a disk to a VM'
date: 2017-10-04 15:07:36.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- PowerCLI
- Scripting
- virtualFrog Posts
- VMware
tags: []
meta:
  _thumbnail_id: '29'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '9963501208'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:56:"https://twitter.com/virtual_dd/status/915564059927433217";}}
  _publicize_done_9340485: '1'
  _wpas_done_9383931: '1'
  publicize_twitter_user: virtual_dd
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2017/10/04/powercli-adding-a-disk-to-a-vm/"
---
Recently I stumbled upon a stubborn vSphere Client at a customer. It just would not let me add a disk a VM. The error message was very generic (and was in the end fixed with a reboot) so I had to find a alternative: Here comes PowerCLI.

<!--more-->

The requirements were as follows:

- size of the disk in GB must be specified
- it should be possible to add a new scsi controller if one chooses to do so
- it should be possible to specify which already present scsi controller should be used
- if no option is specified always add the disk to the last scsi controller

It was easy enough to get the script working. One thing I learned was that "new-scsicontroller" requires the VM to be in the "PoweredOff" State.

Update: this script is now published at [GitHub](https://github.com/virtualFrog/PowerCLI-Scripts).

Here is the script:

[code language="powershell"]  
 Add-DiskToVm.ps1 -vMName reg-belairi81 -vCenter virtualfrogvc.virtual.frog -diskGB 10

This will attach a 10 GB disk to the VM on the last of its scsi controllers  
.EXAMPLE  
PS\> Add-DiskToVm.ps1 -vCenter virtualfrogvc.virtual.frog -vMName reg-belairi81 -diskGB 10 -addController:$true -controllerType paravirtual

This will attach a 10 GB disk to the VM and attach it to a new scsi controller of type paravirtual  
.EXAMPLE  
PS\> Add-DiskToVm.ps1 -vCenter virtualfrogvc.virtual.frog -vMName reg-belairi81 -diskGB 10 -controllerNumber 2

This will attach a 10 GB disk to the VM and attach it to the second controller on the VM  
#\>

##################################################################################  
# Script: Add-DiskToVm.ps1  
# Datum: 04.10.2017  
# Author: Bechtle Steffen Schweiz AG (c) 2017  
# Version: 1.0  
# History: Check VMs current set of SCSI Controllers when adding Disks  
##################################################################################

[CmdletBinding(SupportsShouldProcess=$true)]  
Param(  
[parameter()]  
[string]$vCenter = "virtualfrogvc.virtual.frog",  
# Change to default VM for testing  
[string]$vMName = "reg-belairi82",  
# Change default Disk Size  
[decimal]$diskGB = 2.5,  
# Hardcode the SCSI Controller # (like 3 for the third controller)  
[int]$controllerNumber = 2000,  
# Add new SCSI Controller while you're at it  
[boolean]$addController = $false,  
# Type of SCSI Controller to add (paravirtual|VirtualLsiLogicSAS)  
[string]$controllerType = "paravirtual"

)  
function get-scsiCount ($vm)  
{  
try {  
return ($vm | get-scsicontroller -ErrorAction Stop).count  
}  
catch {  
Write-Host "Could not count Scsi Controller of $vm"  
exit  
}  
}

function get-scsiID ($vm, $number)  
{  
try {  
return ($vm |get-scsicontroller |select -skip ($number-1) -first 1).ID  
}  
catch {  
Write-Host "Could not get scsi controller number $number from $vm"  
exit  
}  
}

function get-scsiType ($vm, $id)  
{  
try {  
return ($vm |get-scsicontroller -ID $id -ErrorAction Stop).Type  
}  
catch {  
Write-Host "Could not determine type of SCSI Controller on vm ($vm)"  
exit  
}  
}

function add-DiskToVmOnController ($vm, $controller)  
{  
try {  
New-Harddisk -Controller $controller -CapacityGB $diskGB -VM $vm -Whatif -ErrorAction Stop  
} catch {  
Write-Host "Could not add disk to VM ($vm)"  
exit  
}  
}

function shutDownVm ($vm)  
{  
try {  
Stop-VMGuest -VM $vm -confirm:$false -ErrorAction Stop  
Write-Host "Successfully send the shutdown command over VMware Tools"  
}  
catch {  
Write-Host -Foregroundcolor:red "The VM did not respond to a VMware tools shutdown command"  
$switch = Read-Host -Prompt "Would you like to Power off the VM $vm ? (yes/no)"  
if ($switch -match "yes") {  
Stop-VM -VM $vm -confirm:$false  
} else {  
Write-Host "You chose not to power off the VM. Stopping the script.."  
exit  
}  
}

while ((get-vm $vMName).PowerState -notmatch "PoweredOff")  
{  
Write-Host "Waiting for $vm to shut down..."  
sleep -s 5  
}  
$vmHasShutDown = $true  
}

####### Main Program ######  
try {  
Import-Module -Name VMware.VimAutomation.Core -ErrorAction Stop | Out-Null  
} catch {  
Write-Host "Could not add VMware PowerCLI Modules"  
exit  
}

try {  
Connect-VIServer $vCenter -WarningAction SilentlyContinue -ErrorAction Stop | Out-Null  
}  
catch {  
Write-Host "Could not connect to vCenter $vCenter"  
exit  
}

Write-Host "Connected to $vCenter. Starting script"

try {  
$vm = Get-VM $vMName -ErrorAction Stop  
} catch {  
Write-Host "Could not find VM with Name $vMName in vCenter $vCenter"  
exit  
}  
if ($addController) {  
if ($vm.PowerState -match "PoweredOn") {  
Write-Host -Foregroundcolor:red "The VM is still powered On."  
$switch = Read-Host -Prompt "Would you like to shut down the VM ($vm)? (yes/no)"  
if ($switch -match "yes") {  
shutDownVm $vm

} else {  
Write-Host "You chose not to shutdown the VM ($vm). Stopping the script now"  
exit  
}  
}  
try {  
$vm |New-Harddisk -CapacityGB $diskGB |new-scsicontroller -type $controllerType -ErrorAction Stop  
if ($vmHasShutDown) {  
$switch = Read-Host -Prompt "The VM was shut down for this operation. Power it back on? (yes/no)"  
if ($switch -match "yes") {  
Start-VM $vm -confirm:$false  
}  
}  
} catch {  
Write-Host "could not add scsi controller with new disk to $vm"  
exit  
}  
} elseif ($controllerNumber -ne 2000) {  
$numberOfControllers = get-scsiCount $vm  
if ($numberOfControllers -gt $controllerNumber) {  
Write-Host "You specified controller number $controllerNumber but the VM ($vm) only has $numberOfControllers controllers"  
exit  
} else {  
$scsiID = get-scsiID $vm $controllerNumber  
Write-Host "The VM ($vm) has $numberOfControllers SCSI Controller(s) attached. You chose to attach a new disk to the $controllerNumber. adapter"

Write-Host "The VM ($vm) has a "(get-scsiType $vm $scsiID)" Controller for the number you provided"  
add-DiskToVmOnController $vm ($vm | get-scsicontroller -ID $scsiID)  
Write-Host "Added a disk of $diskGB GB to $vm on controller "($vm | get-scsicontroller -ID $scsiID).Name  
}  
}  
else {  
$numberOfControllers = get-scsiCount $vm  
$scsiID = get-scsiID $vm $numberOfControllers  
Write-Host "The VM ($vm) has $numberOfControllers SCSI Controller(s) attached"

Write-Host "The VM ($vm) has a "(get-scsiType $vm $scsiID)" Controller as the last one"  
add-DiskToVmOnController $vm ($vm | get-scsicontroller -ID $scsiID)  
Write-Host "Added a disk of $diskGB GB to $vm on controller "($vm | get-scsicontroller -ID $scsiID).Name  
}  
[/code]

