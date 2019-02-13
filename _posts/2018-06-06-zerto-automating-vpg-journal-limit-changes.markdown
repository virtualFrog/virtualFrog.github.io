---
author: virtualfrog
comments: true
date: 2018-06-06 09:20:54+00:00
layout: post
link: https://virtualfrog.wordpress.com/2018/06/06/zerto-automating-vpg-journal-limit-changes/
slug: zerto-automating-vpg-journal-limit-changes
title: 'Zerto: Automating VPG journal limit changes'
wordpress_id: 734
categories:
- General
- Zerto
---

I helped one of our customers implement Zerto in their environment. When you deploy Zerto and start protecting your VMs with it you need to know which applications you want to group together into virtual protection groups (VPGs). Another thing you'd define beforehand would be how many days you would want to be able to go back in time with Zerto and use that as a base to define the journal size limit of your VPGs.

<!-- more -->

Now, some VMs produce a lot of I/Os while other are basically silent. This is key in how much space they are going to use and need in their VPG. So while for most of your VPGs the intially defined limit will be sufficient to reach your desired amount of time to go back it will not nearly be enough for your most I/O intensive workloads.

When you then go ahead and change the journal limit on this VPG it will NOT update this setting on VMs that are already inside this VPG. When we first noticed this we thought this must be a bug and opened a case with Zerto. The support engineer then told us that this is not a bug but rather it's a feature... The customer and we had a different view on that so we started looking at the zerto API to remedy this unsatisfying situation.

Long story short: Below is a script that will enable you to change the journal limit on a VPG and then iterate through all VMs in that VPG and set the same values (makes perfect sense in our use-case).

Update: this script is now published at [GitHub](https://github.com/virtualFrog/PowerCLI-Scripts).

[code language="powershell"]

param(
[Parameter(Mandatory = $true)][string]$zvm = "zvm.virtualfrog.wordpress.com",
[Parameter(Mandatory = $true)][string]$vpgName = "Test",
[Parameter(Mandatory = $true)][Int32]$LimitValueInGB = 300,
[Parameter(Mandatory = $true)][Int32]$ThresholdValueInGB = 250
)

#----------------------------------------------------------[Declarations]----------------------------------------------------------

# Build Base URL -> for all RestMethods
$baseURL = "https://" + $zvm + ":443/v1/"

# New Limit Value in MB
$LimitValueInMB = $LimitValueInGB * 1024

# New Warning Threshold in MB
$ThresholdValueInMB = $ThresholdValueInGB * 1024

# Responsetype
$TypeJSON = "application/json"

#-----------------------------------------------------------[Functions]------------------------------------------------------------

Function Get-ZertoSession ($ZertoUser, $ZertoPassword) {
# Authenticating with Zerto APIs
$xZertoSessionURL = $baseURL + "session/add"
$authInfo = ("{0}:{1}" -f $ZertoUser, $ZertoPassword)
$authInfo = [System.Text.Encoding]::UTF8.GetBytes($authInfo)
$authInfo = [System.Convert]::ToBase64String($authInfo)
$headers = @{Authorization = ("Basic {0}" -f $authInfo)}
$sessionBody = '{"AuthenticationMethod": "1"}'

# Get Zerto Session Response
$xZertoSessionResponse = Invoke-WebRequest -Uri $xZertoSessionURL -Headers $headers -Method POST -Body $sessionBody -ContentType $TypeJSON

# Extracting x-zerto-session from the response
$xZertoSession = $xZertoSessionResponse.headers.get_item("x-zerto-session")

# Return Zerto Session Header
return @{"Accept" = "application/json"; "x-zerto-session" = $xZertoSession}
}

#-----------------------------------------------------------[Execution]------------------------------------------------------------

# Get Zerto Session
$zertoSessionHeader = Get-ZertoSession -ZertoUser "virtualFrog" -ZertoPassword "Password"

# Get VPG
$vpgListUrl = $baseURL + "vpgs"
$vpg = Invoke-RestMethod -Uri $vpgListUrl -Headers $zertoSessionHeader -ContentType $TypeJSON

#Filter the one from the paramater
$vpgToEdit = $vpg |Where-Object {$_.VpgName -eq "$vpgName"}
Write-Host "Changing Journal Limit and Threshold Settings of: "($vpgToEdit.VpgName)

#Get the identifier of this VPG
$VPGidentifier = $vpgToEdit.VpgIdentifier
$VPGidentifierJSON = '{"VpgIdentifier":"' + $VPGidentifier + '"}'
#$VPGidentifier

# Get VPG Settings Identifier
$VPGSettingsIDURL = $baseURL + "vpgSettings"
$VPGSettingsIdentifier = Invoke-RestMethod -Method Post -Uri $VPGSettingsIDURL -Body $VPGidentifierJSON -ContentType $TypeJSON -Headers $zertoSessionHeader

# Set VPG Settings URL
$VPGSettingsURL = $baseURL + "vpgSettings/" + $VPGSettingsIdentifier
$VPGSettingsBasicURL = $VPGSettingsURL + "/basic"
$VPGSettingsJournalURL = $VPGSettingsURL + "/journal"
$VPGSettingsCommitURL = $VPGSettingsURL + "/commit"
$VPGSettingsVMsJournalURL = $VPGSettingsURL + "/vms"

# Get the actual VPG Settings
$VPGSettings = Invoke-RestMethod -Uri $VPGSettingsURL -Headers $zertoSessionHeader -ContentType $TypeJSON

$HardLimitGB = $VPGSettings.Journal.Limitation.HardLimitInMB / 1024
$WarningThresholdGB = $VPGSettings.Journal.Limitation.WarningThresholdInMB / 1024
#Write-Host "Hard Limit of this VPG in GB: " $HardLimitGB
#Write-Host "Warning Threshold of this VPG in GB: " $WarningThresholdGB

#Change Setting of VPG
$VPGSettings.Journal.Limitation.HardLimitInMB = $LimitValueInMB
$VPGSettings.Journal.Limitation.WarningThresholdInMB = $ThresholdValueInMB

$data = @{Limitation = @{
HardLimitInMB = $LimitValueInMB
WarningThresholdInMB = $ThresholdValueInMB
}
}
$json = $data | ConvertTo-Json

$vmdata = @{Journal = @{
Limitation = @{
HardLimitInMB = $LimitValueInMB
WarningThresholdInMB = $ThresholdValueInMB
}
}
}
$vmjson = $vmdata |ConvertTo-Json
#Write Change to Zerto
$ChangedVPG = Invoke-RestMethod -Uri $VPGSettingsJournalURL -Method Put -Body $json -Headers $zertoSessionHeader -ContentType $TypeJSON

foreach ($vm in $VPGSettings.VMs) {
$vmIdentifier = $vm.VmIdentifier
$VPGSettingsVMsJournalURL = $VPGSettingsURL + "/vms/" + $vmIdentifier
$VMHardLimitGB = $vm.Journal.Limitation.HardLimitInMB / 1024
$VMWarningThreshold = $vm.Journal.Limitation.WarningThresholdInMB / 1024

#Write-Host "VM ("$vm.VmIdentifier") has Limit of "$VMHardLimitGB " GB and Warning of " $VMWarningThreshold "GB"
$ChangedVPG = Invoke-RestMethod -Uri $VPGSettingsVMsJournalURL -Method Put -Body $vmjson -Headers $zertoSessionHeader -ContentType $TypeJSON

}

$VPGCommit = Invoke-RestMethod -Method Post -Uri $VPGSettingsCommitURL -ContentType $TypeJSON -Headers $zertoSessionHeader
Write-Host -ForegroundColor Green "Changed the valued successfully!"

[/code]
