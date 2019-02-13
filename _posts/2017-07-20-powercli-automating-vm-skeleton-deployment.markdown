---
author: virtualfrog
comments: true
date: 2017-07-20 11:11:52+00:00
layout: post
link: https://virtualfrog.wordpress.com/2017/07/20/powercli-automating-vm-skeleton-deployment/
slug: powercli-automating-vm-skeleton-deployment
title: 'PowerCLI: Automating VM Skeleton Deployment'
wordpress_id: 241
categories:
- General
- Scripting
- VMware
tags:
- PowerCLI
---

We recently had to write a script for a customer which would deploy a VM skeleton. The script would be called from an automated solution the customer was building and had the following requirements:



	
  * Use given service-account for interaction in vCenter Server

	
  * Do different configuration based on the vm name which is passed as a parameter

	
    * test servers need to be placed in the "test" resource pool

	
    * test servers get a thin provisioned disk

	
    * production server need to be placed in "production" resource pool

	
    * production server need to placed in "test" resource pool




	
  * return value needs to be the MAC address of the created VM

	
  * VM must not exceed two vCPUs

	
  * VM must no exceed 8 GB RAM

	
  * VM disk must not be greater than 100 GB


<!-- more -->


## Getting the authentication right: VICredentialStore


In order to be able to use the service account we had to find a way to authenticate the user in a secure way without user interaction. this is the script which creates a file with the given information:



	
  * host (vCenter) to log on to

	
  * username

	
  * password


Update: this script is now published at [GitHub](https://github.com/virtualFrog/PowerCLI-Scripts).

[code language="powershell"]
# import VMware related modules
Import-Module -Name VMware.VimAutomation.Core -ErrorAction SilentlyContinue | Out-Null

# get server and credential input
$server = Read-Host "Please provide the servername (fqdn)"
$user_name = Read-Host "please provide the username (domain\user)"
$user_password_encrypted = Read-Host "please input the password" -AsSecureString
$file_location = Read-Host "where should the login.creds file be stored? (e.g.:c:\script)"

# convert password back to plain text for further use with New-VICredentialStoreItem
$BSTR = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($user_password_encrypted)
$user_password_decrypted = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($BSTR) 

# create credential file
New-VICredentialStoreItem -Host $server -User $user_name -Password $user_password_decrypted -File "$file_location\login.creds"

# remove VMware related modules
Remove-Module -Name VMware.VimAutomation.Core -ErrorAction SilentlyContinue | Out-Null
[/code]


## Automating the VM creation


In this particular scenario a few things were a given which we then "hard coded" into the script. You can easily customize it in order to have all variables passed as parameters or just change it to the values of your environment.

[code language="powershell"]
############################################################################################
# Script name:     VmAutomatedDeployment.ps1
# Description:     For simple virtual machine container provisioning use only.
# Version:         1.2
# Date:            02.02.2017
# Author:          Bechtle Steffen Schweiz AG
# History:         02.02.2017 - First tested release
#                  19.07.2017 - Replaced Portgroup for Networkname, diskformat and resource pool based on hostname
#                  19.07.2017 - Added return value: MAC Address of created VM as requested by customer
############################################################################################

# Example: # e.g.: .\VmAutomatedDeployment.ps1 -vm_name Test_A053763 -vm_guestid windows9_64Guest -vm_memory 2048 -vm_cpu 2 -vm_network "Server_2037_L2"

param (

    [string]$vm_guestid, # GuestOS identifier from VMware e.g. windows_64Guest for Windows 10 x64
    [string]$vm_name, # Name of the virtual machine
    #[int64]$vm_disk, # System disk size in GB
    [int32]$vm_memory, # Memory size in MB
    [int32]$vm_cpu, # Amount of vCPUs
    [string]$vm_network # Portgroup name to connect
    #[string]$vm_folder # VM Folder location

    # to get the GuestOS identifier run: [VMware.Vim.VirtualMachineGuestOsIdentifier].GetEnumValues()
    # Common Windows versions:
    #-------------------------
    # windows7_64Guest          // Windows 7 (x64)
    # windows7Server64Guest     // Windows Server 2008 R2
    # windows8_64Guest          // Windows 8 (x64)
    # windows8Server64Guest     // Windows Server 2012 R2
    # windows9_64Guest          // Windows 10 (x64)
    # windows9Server64Guest     // Windows Server 2016
)

# clear global ERROR variable
$Error.Clear()

# import vmware related modules, get the credentials and connect to the vCenter server
Import-Module -Name VMware.VimAutomation.Core -ErrorAction SilentlyContinue | Out-Null
Import-Module -Name VMware.VimAutomation.Vds -ErrorAction SilentlyContinue |Out-Null
$creds = Get-VICredentialStoreItem -file  "C:\scripts\VmAutomatedDeployment\login.creds"
Connect-VIServer -Server $creds.Host -User $creds.User -Password $creds.Password |Out-Null

# define global variables
$init_cluster = 'virtualFrogLab'
$init_datastore = 'VF_ds_01'
$vm_folder = 'VM-Staging'
$current_date = $(Get-Date -format "dd.MM.yyyy HH:mm:ss")
$log_file = "C:\Scripts\VmAutomatedDeployment\log_$(Get-Date -format "yyyyMMdd").txt"
[int64]$vm_disk = 72
$dvPortGroup = Get-VDPortgroup -Name $vm_network

# check if var $VM_NAME already exists in the vCenter inventory

$CheckInventoryByVmName = Get-VM -Name $vm_name -ErrorAction Ignore

if ($CheckInventoryByVmName) {

    Write-Host "This virtual machine already exists!"

} else {

    # check the inputs for provisioning sizing of the virtual machine
    # allowed maximums: 2 vCPU / 8192MB vRAM / 100GB vDISK

    if ($vm_cpu -gt 2) {Write-Host "You input is invalid! (max. 2 vCPUs allowed)"}
    elseif ($vm_memory -gt 8192) {Write-Host "You input is invalid! (max. 8GB vRAM allowed)"}
    elseif ($vm_disk -gt 100) {Write-Host "You input is invalid! (max. 100GB vDisk size allowed)"}
    else {

        # create new virtual machine container
        # e.g.: .\VmAutomatedDeployment.ps1 -vm_name Test_A054108 -vm_guestid windows9_64Guest -vm_disk 72 -vm_memory 2048 -vm_cpu 2 -vm_network "Test 10.10.0.0"
        if ($vm_name -like "vm-t*")
        {
            $diskformat = "Thin"
            $init_cluster ="Test"
        }else {
            $diskformat = "Thick"
            $init_cluster = "Production"
        }
        $create_vm = New-VM -Name $vm_name -GuestId $vm_guestid -Location $vm_folder -ResourcePool $init_cluster -Datastore $init_datastore -DiskGB $vm_disk -DiskStorageFormat $diskformat -MemoryMB $vm_memory -NumCpu $vm_cpu -Portgroup $dvPortGroup -CD -Confirm:$false -ErrorAction SilentlyContinue

        # check if virtual machine exists
        if ($create_vm) {
            # change all network adapters to VMXNET3
            $change_vm_network = Get-VM -Name $create_vm | Get-NetworkAdapter | Set-NetworkAdapter -Type Vmxnet3 -Confirm:$false -ErrorAction SilentlyContinue

            # check if network adapter exists
            if ($change_vm_network) {
                Add-Content -Path $log_file -Value "$current_date     SCRIPT          $message"
                $macaddress = (get-vm -Name $create_vm |get-networkadapter).MacAddress
            } else {
                Write-Host "There was an unexpected error during the provisioning. For more information see log file: $log_file"
            }
        } else {
            Write-Host "There was an unexpected error during the provisioning. For more information see log file: $log_file"

        }

    }

}

# cleanup and removal of loaded VMware modules
Disconnect-VIServer -Server $creds.Host -Confirm:$false
Remove-Module -Name VMware.VimAutomation.Core -ErrorAction SilentlyContinue | Out-Null

# write all error messages to the log file
Add-Content -Path $log_file -Value $Error

#return the MAC address
return $macaddress
[/code]
