---
layout: post
title: 'PowerCLI: Reporting the duration of  Snapshots'
date: 2017-10-10 13:11:57.000000000 +02:00
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
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '10166750956'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:56:"https://twitter.com/virtual_dd/status/917709284317782016";}}
  _publicize_done_9340485: '1'
  _wpas_done_9383931: '1'
  publicize_twitter_user: virtual_dd
  publicize_linkedin_url: https://www.linkedin.com/updates?discuss=&scope=391645417&stype=M&topic=6323474976993214464&type=U&a=6-f3
  _publicize_done_14862392: '1'
  _wpas_done_14749148: '1'
  _thumbnail_id: '29'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2017/10/10/powercli-reporting-the-duration-of-snapshots/"
---
One of our customers had a problem with VMs losing network connectivity while the backup was running. Their backup solution was based on VMware snapshots. After the end-users complained about a service not being responsive they investigated and found out that during the creation of the snapshot they lose the pings.

In response they wanted to check how long the snapshots take on all of their VMs. Even though their environment was not very big, it would have been very tedious to gather this information manually. PowerCLI to the rescue!

<!--more-->

The script leverages Luc's awesome Get-TaskPlus function (which can be found [here](http://www.lucd.info/2013/06/01/task-data-mining-an-improved-get-task/)) with a few enhancements:

- The time from the events are converted to local time (instead of UTC)
- Some try/catch to filter exceptions

Then it was just a matter of filtering for the correct task objects and selecting the desired properties. Because the customer wanted to see the time it took for the task to complete I've added a "New-Timespan" object to the select which calculates the total amount of seconds between "Task Started" and "Task completed".

Update: this script is now published at [GitHub](https://github.com/virtualFrog/PowerCLI-Scripts).

Here is the script:

[code language="powershell"]  
# import vmware related modules, get the credentials and connect to the vCenter server  
Import-Module -Name VMware.VimAutomation.Core -ErrorAction SilentlyContinue | Out-Null  
#$creds = Get-VICredentialStoreItem -file "D:\Scripts\CreateSnapshotCreationOverview\login.creds"  
#Connect-VIServer -Server $creds.Host -User $creds.User -Password $creds.Password  
connect-viserver vCenter.virtualfrog.lab

function Get-TaskPlus {

Get-TaskPlus -Start (Get-Date).AddDays(-1)  
.EXAMPLE  
 PS\> Get-TaskPlus -Alarm $alarm -Details  
#\>

param(  
 [CmdletBinding()]  
 [VMware.VimAutomation.ViCore.Impl.V1.Alarm.AlarmDefinitionImpl]$Alarm,  
 [VMware.VimAutomation.ViCore.Impl.V1.Inventory.InventoryItemImpl]$Entity,  
 [switch]$Recurse = $false,  
 [VMware.Vim.TaskInfoState[]]$State,  
 [DateTime]$Start,  
 [DateTime]$Finish,  
 [string]$UserName,  
 [int]$MaxSamples = 100,  
 [switch]$Reverse = $true,  
 [VMware.VimAutomation.ViCore.Impl.V1.VIServerImpl[]]$Server = $global:DefaultVIServer,  
 [switch]$Realtime,  
 [switch]$Details,  
 [switch]$Keys,  
 [int]$WindowSize = 100  
 )

begin {  
 function Get-TaskDetails {  
 param(  
 [VMware.Vim.TaskInfo[]]$Tasks  
 )  
 begin{  
 $psV3 = $PSversionTable.PSVersion.Major -ge 3  
 }

process{  
 $tasks | %{  
 if($psV3){  
 $object = [ordered]@{}  
 }  
 else {  
 $object = @{}  
 }  
 $object.Add("Name",$\_.Name)  
 $object.Add("Description",$\_.Description.Message)  
 if($Details){$object.Add("DescriptionId",$\_.DescriptionId)}  
 if($Details){$object.Add("Task Created",$\_.QueueTime.tolocaltime())}  
 $object.Add("Task Started",$\_.StartTime.tolocaltime())  
 if($Details){$object.Add("Task Ended",$\_.CompleteTime.tolocaltime())}  
 $object.Add("State",$\_.State)  
 $object.Add("Result",$\_.Result)  
 $object.Add("Entity",$\_.EntityName)  
 $object.Add("VIServer",$VIObject.Name)  
 $object.Add("Error",$\_.Error.ocalizedMessage)  
 if($Details){  
 $object.Add("Cancelled",(&{if($\_.Cancelled){"Y"}else{"N"}}))  
 $object.Add("Reason",$\_.Reason.GetType().Name.Replace("TaskReason", ""))  
 $object.Add("AlarmName",$\_.Reason.AlarmName)  
 $object.Add("AlarmEntity",$\_.Reason.EntityName)  
 $object.Add("ScheduleName",$\_.Reason.Name)  
 $object.Add("User",$\_.Reason.UserName)  
 }  
 if($keys){  
 $object.Add("Key",$\_.Key)  
 $object.Add("ParentKey",$\_.ParentTaskKey)  
 $object.Add("RootKey",$\_.RootTaskKey)  
 }

New-Object PSObject -Property $object  
 }  
 }  
 }

$filter = New-Object VMware.Vim.TaskFilterSpec  
 if($Alarm){  
 $filter.Alarm = $Alarm.ExtensionData.MoRef  
 }  
 if($Entity){  
 $filter.Entity = New-Object VMware.Vim.TaskFilterSpecByEntity  
 $filter.Entity.entity = $Entity.ExtensionData.MoRef  
 if($Recurse){  
 $filter.Entity.Recursion = [VMware.Vim.TaskFilterSpecRecursionOption]::all  
 }  
 else{  
 $filter.Entity.Recursion = [VMware.Vim.TaskFilterSpecRecursionOption]::self  
 }  
 }  
 if($State){  
 $filter.State = $State  
 }  
 if($Start -or $Finish){  
 $filter.Time = New-Object VMware.Vim.TaskFilterSpecByTime  
 $filter.Time.beginTime = $Start  
 $filter.Time.endTime = $Finish  
 $filter.Time.timeType = [vmware.vim.taskfilterspectimeoption]::startedTime  
 }  
 if($UserName){  
 $userNameFilterSpec = New-Object VMware.Vim.TaskFilterSpecByUserName  
 $userNameFilterSpec.UserList = $UserName  
 $filter.UserName = $userNameFilterSpec  
 }  
 $nrTasks = 0  
 }

process {  
 foreach($viObject in $Server){  
 $si = Get-View ServiceInstance -Server $viObject  
 $tskMgr = Get-View $si.Content.TaskManager -Server $viObject

if($Realtime -and $tskMgr.recentTask){  
 $tasks = Get-View $tskMgr.recentTask  
 $selectNr = [Math]::Min($tasks.Count,$MaxSamples-$nrTasks)  
 Get-TaskDetails -Tasks[0..($selectNr-1)]  
 $nrTasks += $selectNr  
 }

try {  
 $tCollector = Get-View ($tskMgr.CreateCollectorForTasks($filter))

if($Reverse){  
 $tCollector.ResetCollector()  
 $taskReadOp = $tCollector.ReadPreviousTasks  
 }  
 else{  
 $taskReadOp = $tCollector.ReadNextTasks  
 }  
 do{  
 $tasks = $taskReadOp.Invoke($WindowSize)  
 if(!$tasks){return}  
 $selectNr = [Math]::Min($tasks.Count,$MaxSamples-$nrTasks)  
 Get-TaskDetails -Tasks $tasks[0..($selectNr-1)]  
 $nrTasks += $selectNr  
 }while($nrTasks -lt $MaxSamples)  
 }  
 catch {  
 Write-Host "A error occured in the collector"  
 }  
 }  
 try {  
 $tCollector.DestroyCollector()  
 }  
 catch {  
 Write-Host "The error not letting us destroy the collector"  
 }  
 }  
}  
$start = (Get-Date).AddDays(-30)  
$finish = (Get-Date)  
$output = Get-TaskPlus -Details -MaxSamples 20000000 -Start $start -Finish $finish  
| ? {$\_.Name -match "CreateSnapshot\_Task" -or $\_.Name -match "RemoveSnapshot\_Task"}  
|select Entity, "Task Created", "Task Started", "Task Ended", User, State, Name,  
@{Name="Duration in Seconds"; Expression = {(New-TimeSpan -start $\_."Task Started" -End $\_."Task Ended").TotalSeconds}}  
# create a CSV file with all snapshot related results  
$output | Export-Csv -Path "c:\temp\snapshot\_create\_and\_remove\_times\_last\_month.csv" -NoTypeInformation

# cleanup and removal of loaded VMware modules  
#Disconnect-VIServer -Server $creds.Host -Confirm:$false  
disconnect-viserver vCenter.virtualfrog.lab -confirm:$false  
Remove-Module -Name VMware.VimAutomation.Core -ErrorAction SilentlyContinue | Out-Null  
[/code]

