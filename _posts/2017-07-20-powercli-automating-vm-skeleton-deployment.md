---
layout: post
title: 'PowerCLI: Automating VM Skeleton Deployment'
date: 2017-07-20 13:11:52.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Scripting
- virtualFrog Posts
- VMware
tags:
- PowerCLI
meta:
  _thumbnail_id: '29'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '7287568689'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:56:"https://twitter.com/virtual_dd/status/887993458069262337";}}
  _publicize_done_9340485: '1'
  _wpas_done_9383931: '1'
  publicize_twitter_user: virtual_dd
  publicize_linkedin_url: https://www.linkedin.com/updates?discuss=&scope=391645417&stype=M&topic=6293759149817761792&type=U&a=QZ0P
  _publicize_done_14862392: '1'
  _wpas_done_14749148: '1'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2017/07/20/powercli-automating-vm-skeleton-deployment/"
---
<p>We recently had to write a script for a customer which would deploy a VM skeleton. The script would be called from an automated solution the customer was building and had the following requirements:</p>
<ul>
<li>Use given service-account for interaction in vCenter Server</li>
<li>Do different configuration based on the vm name which is passed as a parameter
<ul>
<li>test servers need to be placed in the "test" resource pool</li>
<li>test servers get a thin provisioned disk</li>
<li>production server need to be placed in "production" resource pool</li>
<li>production server need to placed in "test" resource pool</li>
</ul>
</li>
<li>return value needs to be the MAC address of the created VM</li>
<li>VM must not exceed two vCPUs</li>
<li>VM must no exceed 8 GB RAM</li>
<li>VM disk must not be greater than 100 GB</li>
</ul>
<p><!--more--></p>
<h2>Getting the authentication right: VICredentialStore</h2>
<p>In order to be able to use the service account we had to find a way to authenticate the user in a secure way without user interaction. this is the script which creates a file with the given information:</p>
<ul>
<li>host (vCenter) to log on to</li>
<li>username</li>
<li>password</li>
</ul>
<p>Update: this script is now published at <a href="https://github.com/virtualFrog/PowerCLI-Scripts" target="_blank" rel="noopener">GitHub</a>.</p>
<p>[code language="powershell"]<br />
# import VMware related modules<br />
Import-Module -Name VMware.VimAutomation.Core -ErrorAction SilentlyContinue | Out-Null</p>
<p># get server and credential input<br />
$server = Read-Host "Please provide the servername (fqdn)"<br />
$user_name = Read-Host "please provide the username (domain\user)"<br />
$user_password_encrypted = Read-Host "please input the password" -AsSecureString<br />
$file_location = Read-Host "where should the login.creds file be stored? (e.g.:c:\script)"</p>
<p># convert password back to plain text for further use with New-VICredentialStoreItem<br />
$BSTR = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($user_password_encrypted)<br />
$user_password_decrypted = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($BSTR) </p>
<p># create credential file<br />
New-VICredentialStoreItem -Host $server -User $user_name -Password $user_password_decrypted -File "$file_location\login.creds"</p>
<p># remove VMware related modules<br />
Remove-Module -Name VMware.VimAutomation.Core -ErrorAction SilentlyContinue | Out-Null<br />
[/code]</p>
<h2>Automating the VM creation</h2>
<p>In this particular scenario a few things were a given which we then "hard coded" into the script. You can easily customize it in order to have all variables passed as parameters or just change it to the values of your environment.</p>
<p>[code language="powershell"]<br />
############################################################################################<br />
# Script name:     VmAutomatedDeployment.ps1<br />
# Description:     For simple virtual machine container provisioning use only.<br />
# Version:         1.2<br />
# Date:            02.02.2017<br />
# Author:          Bechtle Steffen Schweiz AG<br />
# History:         02.02.2017 - First tested release<br />
#                  19.07.2017 - Replaced Portgroup for Networkname, diskformat and resource pool based on hostname<br />
#                  19.07.2017 - Added return value: MAC Address of created VM as requested by customer<br />
############################################################################################</p>
<p># Example: # e.g.: .\VmAutomatedDeployment.ps1 -vm_name Test_A053763 -vm_guestid windows9_64Guest -vm_memory 2048 -vm_cpu 2 -vm_network "Server_2037_L2"</p>
<p>param (</p>
<p>    [string]$vm_guestid, # GuestOS identifier from VMware e.g. windows_64Guest for Windows 10 x64<br />
    [string]$vm_name, # Name of the virtual machine<br />
    #[int64]$vm_disk, # System disk size in GB<br />
    [int32]$vm_memory, # Memory size in MB<br />
    [int32]$vm_cpu, # Amount of vCPUs<br />
    [string]$vm_network # Portgroup name to connect<br />
    #[string]$vm_folder # VM Folder location</p>
<p>    # to get the GuestOS identifier run: [VMware.Vim.VirtualMachineGuestOsIdentifier].GetEnumValues()<br />
    # Common Windows versions:<br />
    #-------------------------
  
 # windows7\_64Guest // Windows 7 (x64)  
 # windows7Server64Guest // Windows Server 2008 R2  
 # windows8\_64Guest // Windows 8 (x64)  
 # windows8Server64Guest // Windows Server 2012 R2  
 # windows9\_64Guest // Windows 10 (x64)  
 # windows9Server64Guest // Windows Server 2016  
)

# clear global ERROR variable  
$Error.Clear()

# import vmware related modules, get the credentials and connect to the vCenter server  
Import-Module -Name VMware.VimAutomation.Core -ErrorAction SilentlyContinue | Out-Null  
Import-Module -Name VMware.VimAutomation.Vds -ErrorAction SilentlyContinue |Out-Null  
$creds = Get-VICredentialStoreItem -file "C:\scripts\VmAutomatedDeployment\login.creds"  
Connect-VIServer -Server $creds.Host -User $creds.User -Password $creds.Password |Out-Null

# define global variables  
$init\_cluster = 'virtualFrogLab'  
$init\_datastore = 'VF\_ds\_01'  
$vm\_folder = 'VM-Staging'  
$current\_date = $(Get-Date -format "dd.MM.yyyy HH:mm:ss")  
$log\_file = "C:\Scripts\VmAutomatedDeployment\log\_$(Get-Date -format "yyyyMMdd").txt"  
[int64]$vm\_disk = 72  
$dvPortGroup = Get-VDPortgroup -Name $vm\_network

# check if var $VM\_NAME already exists in the vCenter inventory

$CheckInventoryByVmName = Get-VM -Name $vm\_name -ErrorAction Ignore

if ($CheckInventoryByVmName) {

Write-Host "This virtual machine already exists!"

} else {

# check the inputs for provisioning sizing of the virtual machine  
 # allowed maximums: 2 vCPU / 8192MB vRAM / 100GB vDISK

if ($vm\_cpu -gt 2) {Write-Host "You input is invalid! (max. 2 vCPUs allowed)"}  
 elseif ($vm\_memory -gt 8192) {Write-Host "You input is invalid! (max. 8GB vRAM allowed)"}  
 elseif ($vm\_disk -gt 100) {Write-Host "You input is invalid! (max. 100GB vDisk size allowed)"}  
 else {

# create new virtual machine container  
 # e.g.: .\VmAutomatedDeployment.ps1 -vm\_name Test\_A054108 -vm\_guestid windows9\_64Guest -vm\_disk 72 -vm\_memory 2048 -vm\_cpu 2 -vm\_network "Test 10.10.0.0"  
 if ($vm\_name -like "vm-t\*")  
 {  
 $diskformat = "Thin"  
 $init\_cluster ="Test"  
 }else {  
 $diskformat = "Thick"  
 $init\_cluster = "Production"  
 }  
 $create\_vm = New-VM -Name $vm\_name -GuestId $vm\_guestid -Location $vm\_folder -ResourcePool $init\_cluster -Datastore $init\_datastore -DiskGB $vm\_disk -DiskStorageFormat $diskformat -MemoryMB $vm\_memory -NumCpu $vm\_cpu -Portgroup $dvPortGroup -CD -Confirm:$false -ErrorAction SilentlyContinue

# check if virtual machine exists  
 if ($create\_vm) {  
 # change all network adapters to VMXNET3  
 $change\_vm\_network = Get-VM -Name $create\_vm | Get-NetworkAdapter | Set-NetworkAdapter -Type Vmxnet3 -Confirm:$false -ErrorAction SilentlyContinue

# check if network adapter exists  
 if ($change\_vm\_network) {  
 Add-Content -Path $log\_file -Value "$current\_date SCRIPT $message"  
 $macaddress = (get-vm -Name $create\_vm |get-networkadapter).MacAddress  
 } else {  
 Write-Host "There was an unexpected error during the provisioning. For more information see log file: $log\_file"  
 }  
 } else {  
 Write-Host "There was an unexpected error during the provisioning. For more information see log file: $log\_file"

}

}

}

# cleanup and removal of loaded VMware modules  
Disconnect-VIServer -Server $creds.Host -Confirm:$false  
Remove-Module -Name VMware.VimAutomation.Core -ErrorAction SilentlyContinue | Out-Null

# write all error messages to the log file  
Add-Content -Path $log\_file -Value $Error

#return the MAC address  
return $macaddress  
[/code]

