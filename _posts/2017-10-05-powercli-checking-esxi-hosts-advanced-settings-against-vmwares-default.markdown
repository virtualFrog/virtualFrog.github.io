---
author: virtualfrog
comments: true
date: 2017-10-05 09:47:03+00:00
layout: post
link: https://virtualfrog.wordpress.com/2017/10/05/powercli-checking-esxi-hosts-advanced-settings-against-vmwares-default/
slug: powercli-checking-esxi-hosts-advanced-settings-against-vmwares-default
title: 'PowerCLI:  Checking ESXi Hosts Advanced Settings against VMware''s default'
wordpress_id: 695
categories:
- General
- PowerCLI
- Scripting
- VMware
---

Over the years and releases from VMware the best practices change sometimes. In the early days we had to set a lot of advanced settings to get a desired feature to run as it should or to make the VMware Admin's life a little easier.

The thing about these settings is that you should check them with every release. Are they still required? Have they been deprecated? Is the behaviour still the same when they get set?

<!-- more -->

To make these checks a little easier I've written a small script. If one wants to check advanced settings one could use the "Get-Advancedsetting" cmdlet. However I found that this cmdlet does not provide the default values of the settings. It only returns the name and the current value. So I took the approach of getting the advanced setting over a esxcli instance. ESXCLI provides that valuable information about the default values (string or int).

So I take in all the advanced settings of a host. I then filter out some settings that I don't want to pop up in the report because I know why I set them and want to keep them. Then each setting's value is compared to it's default value. If there is a match, it too will get filtered out.

At the end of the script I will have a list of settings of all my hosts which are not on their default value. I can then go ahead and check those settings in the documentation and change their value back to default (if I don't need them anymore) or I could add them to my filter if I want them to stay that way.

Update: this script is now published at [GitHub](https://github.com/virtualFrog/PowerCLI-Scripts).

Here is the script:

[code language="powershell"]
 .\Get-AdvancedSettingsNotDefault.ps1 -hostName esx01.virtualfrog.lab -delta:$true

	Description
	-----------
	Gets All settings that are not at default value from given host
.EXAMPLE
    C:\foo> .\Get-AdvancedSettingsNotDefault.ps1 -delta:$true

    Description
    -----------
    Gets All settings from all hosts that are not at default value
.EXAMPLE
    C:\foo> .\Get-AdvancedSettingsNotDefault.ps1 -delta:$false

    Description
    -----------
    Gets All settings from all hosts
#> 

param (
	[Parameter(Mandatory=$False)]
	[string]$hostName="none",
    [Parameter(Mandatory=$False)]
    [boolean]$delta=$false
)

$excludedSettings = "/Migrate/Vmknic|/UserVars/ProductLockerLocation|/UserVars/SuppressShellWarning"
$AdvancedSettings = @()
$AdvancedSettingsFiltered = @()

# Checking if host exists
if ($hostName -ne "none") {
   try {
        $vmhost = Get-VMHost $hostName -ErrorAction Stop
    } catch {
        Write-Host -ForegroundColor Red "There is no host available with name" $hostName
        exit
    } 

    # Retrieving advanced settings
    $esxcli = $vmhost | get-esxcli -V2
    $AdvancedSettings = $esxcli.system.settings.advanced.list.Invoke(@{delta = $delta}) |select @{Name="Hostname"; Expression = {$vmhost}},Path,DefaultIntValue,IntValue,DefaultStringValue,StringValue,Description

    # Displaying results
    #$AdvancedSettings

} else {
    $vmhosts = get-vmhost
    foreach ($vmhost in $vmhosts) {
        $esxcli = $vmhost | get-esxcli -V2
        $AdvancedSettings += $esxcli.system.settings.advanced.list.Invoke(@{delta = $delta}) |select @{Name="Hostname"; Expression = {$vmhost}},Path,DefaultIntValue,IntValue,DefaultStringValue,StringValue,Description
    }
    #$AdvancedSettings
}

# Browsing advanced settings and check for mismatch
ForEach ($advancedSetting in $AdvancedSettings.GetEnumerator()) {
    if ( ($AdvancedSetting.Path -notmatch $excludedSettings) -And (($AdvancedSetting.IntValue -ne $AdvancedSetting.DefaultIntValue) -Or ($AdvancedSetting.StringValue -notmatch $AdvancedSetting.DefaultStringValue) ) ){
        $line = "" | Select Hostname,Path,DefaultIntValue,IntValue,DefaultStringValue,StringValue,Description
        $line.Hostname = $advancedSetting.Hostname
        $line.Path = $advancedSetting.Path
        $line.DefaultIntValue = $advancedSetting.DefaultIntValue
        $line.IntValue = $advancedSetting.IntValue
        $line.DefaultStringValue = $advancedSetting.DefaultStringValue
        $line.StringValue = $advancedSetting.StringValue
        $line.Description = $advancedSetting.Description
        $AdvancedSettingsFiltered += $line
    }
}
$AdvancedSettingsFiltered
[/code]
