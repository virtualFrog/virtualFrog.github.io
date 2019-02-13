---
author: virtualfrog
comments: true
date: 2018-06-05 09:15:04+00:00
layout: post
link: https://virtualfrog.wordpress.com/2018/06/05/powercli-change-admission-control-percentages/
slug: powercli-change-admission-control-percentages
title: 'PowerCLI: Change Admission Control Percentages'
wordpress_id: 732
categories:
- General
- PowerCLI
- Scripting
- VMware
---

I was recently asked by a customer to explain some of the vSphere 6.5 features that would be interesting for his environment. I knew the environment so I pointed out some things that 6.5 does better than the currently installed 6.0. One of those things I mentioned was the automated adjustment of the resource percentage in the HA admission control settings.

<!-- more -->

For those of you who don't know: vSphere 6.5 does automatically change the resource percentage when you add or remove hosts to a cluster. For example: You have a 4-Node cluster and set the HA admission control setting to cluster resource percentage and you want to tolerate 1 host failure. vSphere sets the percentages to 25% for CPU and Memory. If you now go ahead and add one host those settings are automatically updated to 20% memory and cpu.

Anyways, the customer wanted that feature because he does that quite often (adding/removing hosts) but he could not update to 6.5 for a couple of reasons (Hardware compatibility and 3rd party software that integrates into vCenter, etc). So he asked me if there was a way to get that feature without updating. My response was: "Everything is possible with PowerCLI".

So, long story, here is the script to backport this 6.5 feature to 6.0. Now, you'd have to call this script every time a change of hosts has occurred in your clusters, or you could let it run twice a day or something (still not as elegant as the 6.5 feature, but close enough).

Update: this script is now published at [GitHub](https://github.com/virtualFrog/PowerCLI-Scripts).

Oh, almost forgot: This script uses the [PSLogging](https://www.powershellgallery.com/packages/PSLogging/2.5.2) Module that is available from the powershellgallery.

[code language="powershell"]

#---------------------------------------------------------[Script Parameters]------------------------------------------------------

Param (
    #Script parameters go here
    [Parameter(Mandatory = $true)][string]$vCenter,
    [Parameter(Mandatory = $true)][string]$cluster,
    [Parameter(Mandatory = $true)][string]$failuresToTolerate
)

#---------------------------------------------------------[Initialisations]--------------------------------------------------------

#Set Error Action to Silently Continue
$ErrorActionPreference = 'SilentlyContinue'

#Import Modules & Snap-ins
Import-Module PSLogging

#----------------------------------------------------------[Declarations]----------------------------------------------------------

#Script Version
$sScriptVersion = '1.0'

#Log File Info
$sLogPath = 'C:\Temp'
$sLogName = 'HA_AdmissionControlLog.log'
$sLogFile = Join-Path -Path $sLogPath -ChildPath $sLogName

#-----------------------------------------------------------[Functions]------------------------------------------------------------

Function Connect-VMwareServer {
    Param ([Parameter(Mandatory = $true)][string]$VMServer)

    Begin {
        Write-LogInfo -LogPath $sLogFile -Message "Connecting to vCenter Server [$VMServer]..."
    }

    Process {
        Try {
            $oCred = Get-Credential -Message 'Enter credentials to connect to vCenter Server'
            Connect-VIServer -Server $VMServer -Credential $oCred -ErrorAction Stop
        }

        Catch {
            Write-LogError -LogPath $sLogFile -Message $_.Exception -ExitGracefully
            Break
        }
    }

    End {
        If ($?) {
            Write-LogInfo -LogPath $sLogFile -Message 'Completed Successfully.'
            Write-LogInfo -LogPath $sLogFile -Message ' '
        }
    }
}

Function checkRAMCompliance {
    Param ([Parameter(Mandatory = $true)][string]$clusterName)
    Begin {
        Write-LogInfo -LogPath $sLogFile -Message "Checking if all hosts in cluster [$clusterName] have the same amount of RAM..."
        $myHostsRAM = @()
    }
    Process {
        Try {
            $clusterObject = Get-Cluster -Name $clusterName -ErrorAction Stop
            $hostObjects = $clusterObject | Get-VMHost -ErrorAction Stop

            foreach ($hostObject in $hostObjects) {

                $HostInfoMEM = "" | Select-Object MEM

                $HostInfoMEM.MEM = $hostObject.MemoryTotalGB

                $myHostsRAM += $HostInfoMEM
            }
            #Check if all values are correct: return true, otherwise return false
            if (@($myHostsRAM | Select-Object -Unique).Count -eq 1) {
                #CPU are the same
                Write-LogInfo -LogPath $sLogFile -Message "RAM in the cluster is the same"
                return $true
            }
            else {
                Write-LogInfo -LogPath $sLogFile -Message "RAM in the cluster is NOT the same"
                return $false
            }

        }
        Catch {
            Write-LogError -LogPath $sLogFile -Message $_.Exception -ExitGracefully
            Break
        }
    }
    End {
        If ($?) {
            Write-LogInfo -LogPath $sLogFile -Message 'Completed Successfully.'
            Write-LogInfo -LogPath $sLogFile -Message ' '
        }
    }
}

Function checkCPUCompliance {
    Param ([Parameter(Mandatory = $true)][string]$clusterName)
    Begin {
        Write-LogInfo -LogPath $sLogFile -Message "Checking if all hosts in cluster [$clusterName] have the same CPU..."
        $myHostsCPU = @()
    }
    Process {
        Try {
            $clusterObject = Get-Cluster -Name $clusterName -ErrorAction Stop
            $hostObjects = $clusterObject | Get-VMHost -ErrorAction Stop

            foreach ($hostObject in $hostObjects) {
                $HostInfoCPU = "" | Select-Object CPU

                $HostInfoCPU.CPU = $hostObject.CpuTotalMhz

                $myHostsCPU += $HostInfoCPU
            }
            #Check if all values are correct: return true, otherwise return false

            if (@($myHostsCPU | Select-Object -Unique).Count -eq 1) {
                Write-LogInfo -LogPath $sLogFile -Message "CPU in the cluster is the same"
                return $true
            }
            else {
                Write-LogInfo -LogPath $sLogFile -Message "CPU in the cluster is NOT the same"
                return $false
            }

        }
        Catch {
            Write-LogError -LogPath $sLogFile -Message $_.Exception -ExitGracefully
            Break
        }
    }
    End {
        If ($?) {
            Write-LogInfo -LogPath $sLogFile -Message 'Completed Successfully.'
            Write-LogInfo -LogPath $sLogFile -Message ' '
        }
    }
}

Function getBiggestCPUInCluster {
    Param ([Parameter(Mandatory=$true)][string]$clusterName)
    Begin {
      Write-LogInfo -LogPath $sLogFile -Message "Trying to get the biggest CPU resource in cluster [$clusterName]..."
    }
    Process {
      Try {
        $clusterObject = Get-Cluster $clusterName -ErrorAction Stop
        $hostObjects = $clusterObject | Get-VMHost -ErrorAction Stop
        $cpuResources =  @()
        foreach ($hostObject in $hostObjects) {
            $cpuInfo = "" | Select-Object CPU
            $cpuInfo.CPU = $hostObject.CpuTotalMhz

            $cpuResources += $cpuInfo
        }
        return ($cpuResources | Measure-Object -Maximum)

      }
      Catch {
        Write-LogError -LogPath $sLogFile -Message $_.Exception -ExitGracefully
        Break
      }
    }
    End {
      If ($?) {
        Write-LogInfo -LogPath $sLogFile -Message 'Completed Successfully.'
        Write-LogInfo -LogPath $sLogFile -Message ' '
      }
    }
  }

  Function getBiggestRAMInCluster {
    Param ([Parameter(Mandatory=$true)][string]$clusterName)
    Begin {
      Write-LogInfo -LogPath $sLogFile -Message "Trying to get the biggest RAM resource in cluster [$clusterName]..."
    }
    Process {
      Try {
        $clusterObject = Get-Cluster $clusterName -ErrorAction Stop
        $hostObjects = $clusterObject | Get-VMHost -ErrorAction Stop
        $ramResources =  @()
        foreach ($hostObject in $hostObjects) {
            $ramInfo = "" | Select-Object RAM
            $ramInfo.RAM = $hostObject.MemoryTotalGB

            $ramResources += $ramInfo
        }
        return ($ramResources | Measure-Object -Maximum)

      }
      Catch {
        Write-LogError -LogPath $sLogFile -Message $_.Exception -ExitGracefully
        Break
      }
    }
    End {
      If ($?) {
        Write-LogInfo -LogPath $sLogFile -Message 'Completed Successfully.'
        Write-LogInfo -LogPath $sLogFile -Message ' '
      }
    }
  }

Function SetAdmissionControl {
    Param ([Parameter(Mandatory = $true)][string]$clusterName)
    Begin {
        Write-LogInfo -LogPath $sLogFile -Message "Setting the admission policy on cluster [$clusterName].."
    }
    Process {
        Try {
            $RAMCompliance = checkRAMCompliance $clusterName
            $CPUCompliance = checkCPUCompliance $clusterName
            $totalAmountofHostsInCluster = (Get-Cluster -Name $clusterName -ErrorAction Stop | Get-VMHost -ErrorAction Stop).Count
            if (($RAMCompliance -eq $true) -and ($CPUCompliance -eq $true)) {
                #Same hardware. calculation very simple
                [int]$ramPercentageToReserve = [math]::Round((100 / ($totalAmountofHostsInCluster) * ($failuresToTolerate)), 0)
                [int]$cpuPercentageToReserve = [math]::Round((100 / ($totalAmountofHostsInCluster) * ($failuresToTolerate)), 0)

            }
            if ($CPUCompliance -eq $false) {
                #calculate with different CPU resources but same RAM resources
                #get biggest CPU amount, total amount and number of hosts in cluster
                $biggestCPUValue = getBiggestCPUInCluster $clusterName
                $totalCPUMhz = (Get-Cluster $clusterName).ExtensionData.Summary.TotalCPU

                [int]$cpuPercentageToReserve = [math]::Round(((($biggestCPUValue) * 100) / ($totalCPUMhz)*($failuresToTolerate)),0)
            }
            if ($RAMCompliance -eq $false) {
                #in this case RAM is the decisive factor
                #get biggest RAM amount, total amount and number of hosts in cluster
                $biggestMemoryValue = getBiggestRAMInCluster $clusterName
                $totalMemoryGB = [math]::Round(((Get-Cluster $clusterName).ExtensionData.Summary.TotalMemory /1024 /1024 /1024),0)

                [int]$ramPercentageToReserve = [math]::Round(((($biggestMemoryValue) * 100) / ($totalMemoryGB)*($failuresToTolerate)),0)
            }

            Write-LogInfo -LogPath $sLogFile -Message "CPU Value calculated: [$cpuPercentageToReserve].."
            Write-LogInfo -LogPath $sLogFile -Message "RAM Value calculated: [$ramPercentageToReserve].."

            $spec = New-Object VMware.Vim.ClusterConfigSpecEx
            $spec.dasConfig = New-Object VMware.Vim.ClusterDasConfigInfo
            $spec.dasConfig.AdmissionControlPolicy = New-Object VMware.Vim.ClusterFailoverResourcesAdmissionControlPolicy
            $spec.dasConfig.AdmissionControlEnabled = $true
            $spec.dasConfig.AdmissionControlPolicy.cpuFailoverResourcesPercent = $cpuPercentageToReserve
            $spec.dasConfig.AdmissionControlPolicy.memoryFailoverResourcesPercent = $ramPercentageToReserve

            $clusterObject = Get-Cluster $clusterName
            $clusterView =  Get-View $clusterObject
            $clusterView.ReconfigureComputeResource_Task($spec, $true)

        }
        Catch {
            Write-LogError -LogPath $sLogFile -Message $_.Exception -ExitGracefully
            Break
        }
    }
    End {
        If ($?) {
            Write-LogInfo -LogPath $sLogFile -Message 'Completed Successfully.'
            Write-LogInfo -LogPath $sLogFile -Message ' '
        }
    }
}

#-----------------------------------------------------------[Execution]------------------------------------------------------------

Start-Log -LogPath $sLogPath -LogName $sLogName -ScriptVersion $sScriptVersion
Connect-VMwareServer -VMServer $vCenter
SetAdmissionControl -clusterName $cluster
Stop-Log -LogPath $sLogFile
[/code]
