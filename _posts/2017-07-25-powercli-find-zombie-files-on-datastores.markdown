---
author: virtualfrog
comments: true
date: 2017-07-25 08:46:10+00:00
layout: post
link: https://virtualfrog.wordpress.com/2017/07/25/powercli-find-zombie-files-on-datastores/
slug: powercli-find-zombie-files-on-datastores
title: 'PowerCLI: Find Zombie Files on Datastores'
wordpress_id: 302
categories:
- General
- Scripting
- VMware
tags:
- PowerCLI
---

Everyone who works in our industry is familiar with RVTools. It's a great tool that achieves a lot in short time in terms of infrastructure checks. I would not want to do my job without it.

One customer wanted to have a e-mail report that showed the "zombie" files on their datastores. After a RVTool check they saw that they wasted a lot of space for not removed snapshot delta disks that were not removed by their backup solution. This is a problem we see in almost every environment.

<!-- more -->So we created a script that would report the files that are not associated with the running VMs.

The script takes a long time to check a large environment, so be patient when you use it. Thanks to Luc from the PowerCLI community for helping us get this function a little faster. (http://www.lucd.info/2016/09/13/orphaned-files-revisited/)

Update: this script is now published at [GitHub](https://github.com/virtualFrog/PowerCLI-Scripts).


## The Script


[code language="powershell"]
##################################################################################
# Script:           RmOrphanedFiles.ps1
# Datum:            25.07.2017
# Author:           Bechtle Schweiz AG (c) 2013 - 2017
# Version:          2.1
# History:          Added Comments
#                   Replaced Add-PSSnapin with Module Command
#                   Replaced Get-VmwOrphan Function
##################################################################################

[CmdletBinding(SupportsShouldProcess=$true)]
Param(
  [parameter()]
  [String]$vCenter = "virtualfrogvc.virtual.frog",
  # Change to a SMTP server in your environment
  [string]$SmtpHost = "mail.virtual.frog",
# Change to default email address you want emails to be coming from
    [string]$From = "virtualFrog@virtual.frog",
# Change to default email address you would like to receive emails
    [Array]$To = @("email@email.com","email2@email.com"),
# Change to default Report Filename you like
    [string]$Attachment = "$env:temp\OrphanedFileReport-"+(Get-Date –f "yyyy-MM-dd")+".csv"
)

function Get-VmwOrphan{
 Get-VmwOrphaned -Datastore DS1
.EXAMPLE
  PS> Get-Datastore -Name DS* | Get-VmwOrphaned
#>

  [CmdletBinding()]
  param(
    [parameter(Mandatory=$true,ValueFromPipeline=$true)]
    [PSObject[]]$Datastore
  )

  Begin{
    $flags = New-Object VMware.Vim.FileQueryFlags
    $flags.FileOwner = $true
    $flags.FileSize = $true
    $flags.FileType = $true
    $flags.Modification = $true

    $qFloppy = New-Object VMware.Vim.FloppyImageFileQuery
    $qFolder = New-Object VMware.Vim.FolderFileQuery
    $qISO = New-Object VMware.Vim.IsoImageFileQuery
    $qConfig = New-Object VMware.Vim.VmConfigFileQuery
    $qConfig.Details = New-Object VMware.Vim.VmConfigFileQueryFlags
    $qConfig.Details.ConfigVersion = $true
    $qTemplate = New-Object VMware.Vim.TemplateConfigFileQuery
    $qTemplate.Details = New-Object VMware.Vim.VmConfigFileQueryFlags
    $qTemplate.Details.ConfigVersion = $true
    $qDisk = New-Object VMware.Vim.VmDiskFileQuery
    $qDisk.Details = New-Object VMware.Vim.VmDiskFileQueryFlags
    $qDisk.Details.CapacityKB = $true
    $qDisk.Details.DiskExtents = $true
    $qDisk.Details.DiskType = $true
    $qDisk.Details.HardwareVersion = $true
    $qDisk.Details.Thin = $true
    $qLog = New-Object VMware.Vim.VmLogFileQuery
    $qRAM = New-Object VMware.Vim.VmNvramFileQuery
    $qSnap = New-Object VMware.Vim.VmSnapshotFileQuery

    $searchSpec = New-Object VMware.Vim.HostDatastoreBrowserSearchSpec
    $searchSpec.details = $flags
    $searchSpec.Query = $qFloppy,$qFolder,$qISO,$qConfig,$qTemplate,$qDisk,$qLog,$qRAM,$qSnap
    $searchSpec.sortFoldersFirst = $true
  }

  Process{
    foreach($ds in $Datastore){
      if($ds.GetType().Name -eq "String"){
        $ds = Get-Datastore -Name $ds
        Write-Host "Checking Datastore $ds"
      }

# Only shared VMFS datastore
      if($ds.Type -eq "VMFS" -and $ds.ExtensionData.Summary.MultipleHostAccess){
        Write-Verbose -Message "$(Get-Date)`t$((Get-PSCallStack)[0].Command)`tLooking at $($ds.Name)"

# Define file DB
        $fileTab = @{}

# Get datastore files
        $dsBrowser = Get-View -Id $ds.ExtensionData.browser
        $rootPath = "[" + $ds.Name + "]"
        $searchResult = $dsBrowser.SearchDatastoreSubFolders($rootPath, $searchSpec) | Sort-Object -Property {$_.FolderPath.Length}
        foreach($folder in $searchResult){
          foreach ($file in $folder.File){
            $key = "$($folder.FolderPath)$(if($folder.FolderPath[-1] -eq ']'){' '})$($file.Path)"
            $fileTab.Add($key,$file)

            $folderKey = "$($folder.FolderPath.TrimEnd('/'))"
            if($fileTab.ContainsKey($folderKey)){
              $fileTab.Remove($folderKey)
            }
          }
        }

# Get VM inventory
        Get-VM -Datastore $ds | %{
          $_.ExtensionData.LayoutEx.File | %{
            if($fileTab.ContainsKey($_.Name)){
              $fileTab.Remove($_.Name)
            }
          }
        }

# Get Template inventory
        Get-Template | where {$_.DatastoreIdList -contains $ds.Id} | %{
          $_.ExtensionData.LayoutEx.File | %{
            if($fileTab.ContainsKey($_.Name)){
              $fileTab.Remove($_.Name)
            }
          }
        }

# Remove system files & folders from list
        $systemFiles = $fileTab.Keys | where{$_ -match "] \.|vmkdump"}
        $systemFiles | %{
          $fileTab.Remove($_)
        }

# Organise remaining files
        if($fileTab.Count){
          $fileTab.GetEnumerator() | %{
            $obj = [ordered]@{
              Name = $_.Value.Path
              Folder = $_.Name
              Size = $_.Value.FileSize
              CapacityKB = $_.Value.CapacityKb
              Modification = $_.Value.Modification
              Owner = $_.Value.Owner
              Thin = $_.Value.Thin
              Extents = $_.Value.DiskExtents -join ','
              DiskType = $_.Value.DiskType
              HWVersion = $_.Value.HardwareVersion
            }
            New-Object PSObject -Property $obj
          }
          Write-Verbose -Message "$(Get-Date)`t$((Get-PSCallStack)[0].Command)`tFound orphaned files on $($ds.Name)!"
        }
        else{
          Write-Verbose -Message "$(Get-Date)`t$((Get-PSCallStack)[0].Command)`tNo orphaned files found on $($ds.Name)."
        }
      }
    }
  }
}

Import-Module -Name VMware.VimAutomation.Core -ErrorAction SilentlyContinue | Out-Null
Connect-VIServer $vCenter -WarningAction SilentlyContinue | Out-Null
Write-Host "Connected to $vCenter. Starting script"

$bodyh = (Get-Date –f "yyyy-MM-dd HH:mm:ss") + "  -  the following orphaned files were found on Datastores. `n"
$body = Get-Datastore | Get-VmwOrphan
$body | Export-Csv "$Attachment" -NoTypeInformation -UseCulture
$body = $bodyh + ($body | Out-String)
$subject = "Report - orphaned Files on Datastores for $vCenter"
send-mailmessage -from "$from" -to $to -subject "$subject" -body "$body" -Attachment "$Attachment" -smtpServer $SmtpHost
Disconnect-VIServer -Server $vCenter -Force:$true -Confirm:$false
[/code]
