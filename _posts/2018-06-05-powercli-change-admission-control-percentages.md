---
layout: post
title: 'PowerCLI: Change Admission Control Percentages'
date: 2018-06-05 11:15:04.000000000 +02:00
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
  _thumbnail_id: '29'
  timeline_notification: '1528190127'
  _publicize_job_id: '18625540367'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:57:"https://twitter.com/virtual_dd/status/1003928269492809728";}}
  _publicize_done_9340485: '1'
  _wpas_done_9383931: '1'
  publicize_twitter_user: virtual_dd
  publicize_linkedin_url: https://www.linkedin.com/updates?discuss=&scope=391645417&stype=M&topic=6409693961925070848&type=U&a=ER9F
  _publicize_done_14862392: '1'
  _wpas_done_14749148: '1'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2018/06/05/powercli-change-admission-control-percentages/"
---
<p>I was recently asked by a customer to explain some of the vSphere 6.5 features that would be interesting for his environment. I knew the environment so I pointed out some things that 6.5 does better than the currently installed 6.0. One of those things I mentioned was the automated adjustment of the resource percentage in the HA admission control settings.</p>
<p><!--more--></p>
<p>For those of you who don't know: vSphere 6.5 does automatically change the resource percentage when you add or remove hosts to a cluster. For example: You have a 4-Node cluster and set the HA admission control setting to cluster resource percentage and you want to tolerate 1 host failure. vSphere sets the percentages to 25% for CPU and Memory. If you now go ahead and add one host those settings are automatically updated to 20% memory and cpu.</p>
<p>Anyways, the customer wanted that feature because he does that quite often (adding/removing hosts) but he could not update to 6.5 for a couple of reasons (Hardware compatibility and 3rd party software that integrates into vCenter, etc). So he asked me if there was a way to get that feature without updating. My response was: "Everything is possible with PowerCLI".</p>
<p>So, long story, here is the script to backport this 6.5 feature to 6.0. Now, you'd have to call this script every time a change of hosts has occurred in your clusters, or you could let it run twice a day or something (still not as elegant as the 6.5 feature, but close enough).</p>
<p>Update: this script is now published at <a href="https://github.com/virtualFrog/PowerCLI-Scripts" target="_blank" rel="noopener">GitHub</a>.</p>
<p>Oh, almost forgot: This script uses the <a href="https://www.powershellgallery.com/packages/PSLogging/2.5.2" target="_blank" rel="noopener">PSLogging</a> Module that is available from the powershellgallery.</p>
<p>[code language="powershell"]</p>
<p>#---------------------------------------------------------[Script Parameters]------------------------------------------------------</p>
<p>Param (<br />
    #Script parameters go here<br />
    [Parameter(Mandatory = $true)][string]$vCenter,<br />
    [Parameter(Mandatory = $true)][string]$cluster,<br />
    [Parameter(Mandatory = $true)][string]$failuresToTolerate<br />
)</p>
<p>#---------------------------------------------------------[Initialisations]--------------------------------------------------------</p>
<p>#Set Error Action to Silently Continue<br />
$ErrorActionPreference = 'SilentlyContinue'</p>
<p>#Import Modules &amp; Snap-ins<br />
Import-Module PSLogging</p>
<p>#----------------------------------------------------------[Declarations]----------------------------------------------------------</p>
<p>#Script Version<br />
$sScriptVersion = '1.0'</p>
<p>#Log File Info<br />
$sLogPath = 'C:\Temp'<br />
$sLogName = 'HA_AdmissionControlLog.log'<br />
$sLogFile = Join-Path -Path $sLogPath -ChildPath $sLogName</p>
<p>#-----------------------------------------------------------[Functions]------------------------------------------------------------</p>
<p>Function Connect-VMwareServer {<br />
    Param ([Parameter(Mandatory = $true)][string]$VMServer)</p>
<p>    Begin {<br />
        Write-LogInfo -LogPath $sLogFile -Message "Connecting to vCenter Server [$VMServer]..."<br />
    }</p>
<p>    Process {<br />
        Try {<br />
            $oCred = Get-Credential -Message 'Enter credentials to connect to vCenter Server'<br />
            Connect-VIServer -Server $VMServer -Credential $oCred -ErrorAction Stop<br />
        }</p>
<p>        Catch {<br />
            Write-LogError -LogPath $sLogFile -Message $_.Exception -ExitGracefully<br />
            Break<br />
        }<br />
    }</p>
<p>    End {<br />
        If ($?) {<br />
            Write-LogInfo -LogPath $sLogFile -Message 'Completed Successfully.'<br />
            Write-LogInfo -LogPath $sLogFile -Message ' '<br />
        }<br />
    }<br />
}</p>
<p>Function checkRAMCompliance {<br />
    Param ([Parameter(Mandatory = $true)][string]$clusterName)<br />
    Begin {<br />
        Write-LogInfo -LogPath $sLogFile -Message "Checking if all hosts in cluster [$clusterName] have the same amount of RAM..."<br />
        $myHostsRAM = @()<br />
    }<br />
    Process {<br />
        Try {<br />
            $clusterObject = Get-Cluster -Name $clusterName -ErrorAction Stop<br />
            $hostObjects = $clusterObject | Get-VMHost -ErrorAction Stop</p>
<p>            foreach ($hostObject in $hostObjects) {</p>
<p>                $HostInfoMEM = "" | Select-Object MEM</p>
<p>                $HostInfoMEM.MEM = $hostObject.MemoryTotalGB</p>
<p>                $myHostsRAM += $HostInfoMEM<br />
            }<br />
            #Check if all values are correct: return true, otherwise return false<br />
            if (@($myHostsRAM | Select-Object -Unique).Count -eq 1) {<br />
                #CPU are the same<br />
                Write-LogInfo -LogPath $sLogFile -Message "RAM in the cluster is the same"<br />
                return $true<br />
            }<br />
            else {<br />
                Write-LogInfo -LogPath $sLogFile -Message "RAM in the cluster is NOT the same"<br />
                return $false<br />
            }</p>
<p>        }<br />
        Catch {<br />
            Write-LogError -LogPath $sLogFile -Message $_.Exception -ExitGracefully<br />
            Break<br />
        }<br />
    }<br />
    End {<br />
        If ($?) {<br />
            Write-LogInfo -LogPath $sLogFile -Message 'Completed Successfully.'<br />
            Write-LogInfo -LogPath $sLogFile -Message ' '<br />
        }<br />
    }<br />
}</p>
<p>Function checkCPUCompliance {<br />
    Param ([Parameter(Mandatory = $true)][string]$clusterName)<br />
    Begin {<br />
        Write-LogInfo -LogPath $sLogFile -Message "Checking if all hosts in cluster [$clusterName] have the same CPU..."<br />
        $myHostsCPU = @()<br />
    }<br />
    Process {<br />
        Try {<br />
            $clusterObject = Get-Cluster -Name $clusterName -ErrorAction Stop<br />
            $hostObjects = $clusterObject | Get-VMHost -ErrorAction Stop</p>
<p>            foreach ($hostObject in $hostObjects) {<br />
                $HostInfoCPU = "" | Select-Object CPU</p>
<p>                $HostInfoCPU.CPU = $hostObject.CpuTotalMhz</p>
<p>                $myHostsCPU += $HostInfoCPU<br />
            }<br />
            #Check if all values are correct: return true, otherwise return false</p>
<p>            if (@($myHostsCPU | Select-Object -Unique).Count -eq 1) {<br />
                Write-LogInfo -LogPath $sLogFile -Message "CPU in the cluster is the same"<br />
                return $true<br />
            }<br />
            else {<br />
                Write-LogInfo -LogPath $sLogFile -Message "CPU in the cluster is NOT the same"<br />
                return $false<br />
            }</p>
<p>        }<br />
        Catch {<br />
            Write-LogError -LogPath $sLogFile -Message $_.Exception -ExitGracefully<br />
            Break<br />
        }<br />
    }<br />
    End {<br />
        If ($?) {<br />
            Write-LogInfo -LogPath $sLogFile -Message 'Completed Successfully.'<br />
            Write-LogInfo -LogPath $sLogFile -Message ' '<br />
        }<br />
    }<br />
}</p>
<p>Function getBiggestCPUInCluster {<br />
    Param ([Parameter(Mandatory=$true)][string]$clusterName)<br />
    Begin {<br />
      Write-LogInfo -LogPath $sLogFile -Message "Trying to get the biggest CPU resource in cluster [$clusterName]..."<br />
    }<br />
    Process {<br />
      Try {<br />
        $clusterObject = Get-Cluster $clusterName -ErrorAction Stop<br />
        $hostObjects = $clusterObject | Get-VMHost -ErrorAction Stop<br />
        $cpuResources =  @()<br />
        foreach ($hostObject in $hostObjects) {<br />
            $cpuInfo = "" | Select-Object CPU<br />
            $cpuInfo.CPU = $hostObject.CpuTotalMhz</p>
<p>            $cpuResources += $cpuInfo<br />
        }<br />
        return ($cpuResources | Measure-Object -Maximum)</p>
<p>      }<br />
      Catch {<br />
        Write-LogError -LogPath $sLogFile -Message $_.Exception -ExitGracefully<br />
        Break<br />
      }<br />
    }<br />
    End {<br />
      If ($?) {<br />
        Write-LogInfo -LogPath $sLogFile -Message 'Completed Successfully.'<br />
        Write-LogInfo -LogPath $sLogFile -Message ' '<br />
      }<br />
    }<br />
  }</p>
<p>  Function getBiggestRAMInCluster {<br />
    Param ([Parameter(Mandatory=$true)][string]$clusterName)<br />
    Begin {<br />
      Write-LogInfo -LogPath $sLogFile -Message "Trying to get the biggest RAM resource in cluster [$clusterName]..."<br />
    }<br />
    Process {<br />
      Try {<br />
        $clusterObject = Get-Cluster $clusterName -ErrorAction Stop<br />
        $hostObjects = $clusterObject | Get-VMHost -ErrorAction Stop<br />
        $ramResources =  @()<br />
        foreach ($hostObject in $hostObjects) {<br />
            $ramInfo = "" | Select-Object RAM<br />
            $ramInfo.RAM = $hostObject.MemoryTotalGB</p>
<p>            $ramResources += $ramInfo<br />
        }<br />
        return ($ramResources | Measure-Object -Maximum)</p>
<p>      }<br />
      Catch {<br />
        Write-LogError -LogPath $sLogFile -Message $_.Exception -ExitGracefully<br />
        Break<br />
      }<br />
    }<br />
    End {<br />
      If ($?) {<br />
        Write-LogInfo -LogPath $sLogFile -Message 'Completed Successfully.'<br />
        Write-LogInfo -LogPath $sLogFile -Message ' '<br />
      }<br />
    }<br />
  }</p>
<p>Function SetAdmissionControl {<br />
    Param ([Parameter(Mandatory = $true)][string]$clusterName)<br />
    Begin {<br />
        Write-LogInfo -LogPath $sLogFile -Message "Setting the admission policy on cluster [$clusterName].."<br />
    }<br />
    Process {<br />
        Try {<br />
            $RAMCompliance = checkRAMCompliance $clusterName<br />
            $CPUCompliance = checkCPUCompliance $clusterName<br />
            $totalAmountofHostsInCluster = (Get-Cluster -Name $clusterName -ErrorAction Stop | Get-VMHost -ErrorAction Stop).Count<br />
            if (($RAMCompliance -eq $true) -and ($CPUCompliance -eq $true)) {<br />
                #Same hardware. calculation very simple<br />
                [int]$ramPercentageToReserve = [math]::Round((100 / ($totalAmountofHostsInCluster) * ($failuresToTolerate)), 0)<br />
                [int]$cpuPercentageToReserve = [math]::Round((100 / ($totalAmountofHostsInCluster) * ($failuresToTolerate)), 0)</p>
<p>            }<br />
            if ($CPUCompliance -eq $false) {<br />
                #calculate with different CPU resources but same RAM resources<br />
                #get biggest CPU amount, total amount and number of hosts in cluster<br />
                $biggestCPUValue = getBiggestCPUInCluster $clusterName<br />
                $totalCPUMhz = (Get-Cluster $clusterName).ExtensionData.Summary.TotalCPU</p>
<p>                [int]$cpuPercentageToReserve = [math]::Round(((($biggestCPUValue) * 100) / ($totalCPUMhz)*($failuresToTolerate)),0)<br />
            }<br />
            if ($RAMCompliance -eq $false) {<br />
                #in this case RAM is the decisive factor<br />
                #get biggest RAM amount, total amount and number of hosts in cluster<br />
                $biggestMemoryValue = getBiggestRAMInCluster $clusterName<br />
                $totalMemoryGB = [math]::Round(((Get-Cluster $clusterName).ExtensionData.Summary.TotalMemory /1024 /1024 /1024),0)</p>
<p>                [int]$ramPercentageToReserve = [math]::Round(((($biggestMemoryValue) * 100) / ($totalMemoryGB)*($failuresToTolerate)),0)<br />
            }</p>
<p>            Write-LogInfo -LogPath $sLogFile -Message "CPU Value calculated: [$cpuPercentageToReserve].."<br />
            Write-LogInfo -LogPath $sLogFile -Message "RAM Value calculated: [$ramPercentageToReserve].."</p>
<p>            $spec = New-Object VMware.Vim.ClusterConfigSpecEx<br />
            $spec.dasConfig = New-Object VMware.Vim.ClusterDasConfigInfo<br />
            $spec.dasConfig.AdmissionControlPolicy = New-Object VMware.Vim.ClusterFailoverResourcesAdmissionControlPolicy<br />
            $spec.dasConfig.AdmissionControlEnabled = $true<br />
            $spec.dasConfig.AdmissionControlPolicy.cpuFailoverResourcesPercent = $cpuPercentageToReserve<br />
            $spec.dasConfig.AdmissionControlPolicy.memoryFailoverResourcesPercent = $ramPercentageToReserve</p>
<p>            $clusterObject = Get-Cluster $clusterName<br />
            $clusterView =  Get-View $clusterObject<br />
            $clusterView.ReconfigureComputeResource_Task($spec, $true)</p>
<p>        }<br />
        Catch {<br />
            Write-LogError -LogPath $sLogFile -Message $_.Exception -ExitGracefully<br />
            Break<br />
        }<br />
    }<br />
    End {<br />
        If ($?) {<br />
            Write-LogInfo -LogPath $sLogFile -Message 'Completed Successfully.'<br />
            Write-LogInfo -LogPath $sLogFile -Message ' '<br />
        }<br />
    }<br />
}</p>
<p>#-----------------------------------------------------------[Execution]------------------------------------------------------------
Start-Log -LogPath $sLogPath -LogName $sLogName -ScriptVersion $sScriptVersion  
Connect-VMwareServer -VMServer $vCenter  
SetAdmissionControl -clusterName $cluster  
Stop-Log -LogPath $sLogFile  
[/code]

