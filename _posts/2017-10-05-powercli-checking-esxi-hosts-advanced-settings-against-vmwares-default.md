---
layout: post
title: 'PowerCLI:  Checking ESXi Hosts Advanced Settings against VMware''s default'
date: 2017-10-05 11:47:03.000000000 +02:00
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
  _oembed_2d9d68aa77b1d55ea12d0cb8cb549d2a: "{{unknown}}"
  _oembed_60cc6311bcebf126820187ee9dac2d3f: "{{unknown}}"
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _thumbnail_id: '29'
  _publicize_job_id: '9994859423'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:56:"https://twitter.com/virtual_dd/status/915875980639637504";}}
  _publicize_done_9340485: '1'
  _wpas_done_9383931: '1'
  publicize_twitter_user: virtual_dd
  _oembed_bca4b9af6d56948dbc8dc16d8deb1a33: "{{unknown}}"
  _oembed_9c874e6494dd46006c0526d3a0f39fed: "{{unknown}}"
  _oembed_c9d17401cf86449116356a4f17bbfc7a: "{{unknown}}"
  _oembed_5c8592547e3d0fc67f661d0f88850fc2: "{{unknown}}"
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2017/10/05/powercli-checking-esxi-hosts-advanced-settings-against-vmwares-default/"
---
<p>Over the years and releases from VMware the best practices change sometimes. In the early days we had to set a lot of advanced settings to get a desired feature to run as it should or to make the VMware Admin's life a little easier.</p>
<p>The thing about these settings is that you should check them with every release. Are they still required? Have they been deprecated? Is the behaviour still the same when they get set?</p>
<p><!--more--></p>
<p>To make these checks a little easier I've written a small script. If one wants to check advanced settings one could use the "Get-Advancedsetting" cmdlet. However I found that this cmdlet does not provide the default values of the settings. It only returns the name and the current value. So I took the approach of getting the advanced setting over a esxcli instance. ESXCLI provides that valuable information about the default values (string or int).</p>
<p>So I take in all the advanced settings of a host. I then filter out some settings that I don't want to pop up in the report because I know why I set them and want to keep them. Then each setting's value is compared to it's default value. If there is a match, it too will get filtered out.</p>
<p>At the end of the script I will have a list of settings of all my hosts which are not on their default value. I can then go ahead and check those settings in the documentation and change their value back to default (if I don't need them anymore) or I could add them to my filter if I want them to stay that way.</p>
<p>Update: this script is now published at <a href="https://github.com/virtualFrog/PowerCLI-Scripts" target="_blank" rel="noopener">GitHub</a>.</p>
<p>Here is the script:</p>
<p>[code language="powershell"]<br />
 .\Get-AdvancedSettingsNotDefault.ps1 -hostName esx01.virtualfrog.lab -delta:$true</p>
<p>	Description<br />
	-----------<br />
	Gets All settings that are not at default value from given host<br />
.EXAMPLE<br />
    C:\foo&gt; .\Get-AdvancedSettingsNotDefault.ps1 -delta:$true</p>
<p>    Description<br />
    -----------<br />
    Gets All settings from all hosts that are not at default value<br />
.EXAMPLE<br />
    C:\foo&gt; .\Get-AdvancedSettingsNotDefault.ps1 -delta:$false</p>
<p>    Description<br />
    -----------
  
 Gets All settings from all hosts  
#\> 

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

