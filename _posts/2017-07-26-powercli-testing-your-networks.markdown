---
author: virtualfrog
comments: true
date: 2017-07-26 13:00:30+00:00
layout: post
link: https://virtualfrog.wordpress.com/2017/07/26/powercli-testing-your-networks/
slug: powercli-testing-your-networks
title: 'PowerCLI: Testing your networks'
wordpress_id: 325
categories:
- General
- Scripting
- VMware
tags:
- PowerCLI
---

If you are knee-deep in a troubleshooting session with a customer you want to be able to have some tools ready that do some tests for you. One of the things that in my case has happened quite a lot was that some network were connected but couldn't communicate outside of the host. So in a HPE Blade Enclosure infrastructure this was most likely a forgotten VLAN tag in the server profiles.

In order to quickly eliminate this problem I have worked on a script that helps you test every network in your environment.

<!-- more -->

The original script is from https://virtualdatacave.com and has initially only been focused on testing distributed Switches. Someone in the VMware Code community has asked for this script to be enhanced so it would support standard switches as well. I thought: Challenge accepted.

Porting the script to be able to test standard switches was quite challenging. Because Standard-switches belong to the host a lot of the script's logic had to be changed to be able to test the networks the same way it works for distributed switches.


## Requirements





	
  * Windows VM with UAC disabled running in the environment you want to test

	
  * CSV file with the information about the networks to test




## CSV: Network map


The csv you provide should look something like this:

![Screen Shot 2017-07-26 at 10.28.11](https://virtualfrog.files.wordpress.com/2017/07/screen-shot-2017-07-26-at-10-28-11.png)

Make sure it really is "comma separated" because excel tends to save csv with semicolons instead of commas.

**PortGroup**: The name of the Portgroup to test

**SourceIP**: This is the ip you will assign your test-vm in that portGroup (make sure it's available, no one wants duplicate IPs)

**GatewayIP**: Gateway of the network to test

**SubnetMask**: Subnet mask of the network to test

**TestIP**: which IP should we try to ping while doing a test in that portgroup? You can, of course always use the same ip if all your networks are routed, or like in the example to a subnet internal test

Update: this script is now published at [GitHub](https://github.com/virtualFrog/PowerCLI-Scripts).


## The Script


[code language="powershell"]


param
(
	[Parameter(Mandatory=$true)]
	[string]$clusterName,
	[Parameter(Mandatory=$true)]
	[string]$dvsName,
	[Parameter(Mandatory=$true)]
	[boolean]$isStandard,
	[Parameter(Mandatory=$true)]
	[pscredential]$creds,
	[Parameter(Mandatory=$true)]
	[string]$vmName,
	[Parameter(Mandatory=$true)]
	[string]$csvFile,
	[int]$timesToPing = 1,
	[int]$pingReplyCountThreshold = 1,
	[Parameter(Mandatory=$true)]
	[string]$resultFile
)

#Setup

# Configure internal variables
$trustVMInvokeable = $false #this is to speed development only.  Set to false.
$testResults = @()
$testPortGroupName = "VirtualFrog"
$data = import-csv $csvFile
$cluster = get-cluster $clusterName
$vm = get-vm $vmName

if ($isStandard -eq $false) {
    $dvs = get-vdswitch $dvsName
    $originalVMPortGroup = ($vm | get-Networkadapter)[0].networkname
	if ($originalVMPortGroup -eq "") {
		$originalVMPortGroup = ($vm | get-virtualswitch -name $dvsName |get-virtualportgroup)[0]
		write-host -Foregroundcolor:red "Adding a fantasy Name to $originalVMPortGroup"
	}
} else {
    $originalVMPortGroup = ($vm | get-Networkadapter)[0].networkname
    $temporaryVar = ($vm |get-networkadapter)[0].NetworkName
	if ($originalVMPortGroup -eq "") {
		$originalVMPortGroup = ($vm |get-vmhost |get-virtualswitch -name $dvsName |get-virtualportgroup -Standard:$true)[0]
		write-host -Foregroundcolor:red "Adding a fantasy Name to $originalVMPortGroup"
	}
}
#We'll use this later to reset the VM back to its original network location if it's empty for some reason wel'll populate it with the first portgroup

#Test if Invoke-VMScript works
if(-not $trustVMInvokeable) {
	if (-not (Invoke-VMScript -ScriptText "echo test" -VM $vm -GuestCredential $creds).ScriptOutput -eq "test") {
		write-output "Unable to run scripts on test VM guest OS!"
		return 1
	}
}

#Define Test Functions
function TestPing($ip, $count, $mtuTest) {
	if($mtuTest) {
		$count = 4 #Less pings for MTU test
		$pingReplyCountThreshold = 3 #Require 3 responses for success on MTU test.  Note this scope is local to function and will not impact variable for future run.
		$script = "ping -f -l 8972 -n $count $ip"
	} else {
		$script =  "ping -n $count $ip"
	}

	write-host "Script to run: $script"
	$result = Invoke-VMScript -ScriptText $script -VM $vm -GuestCredential $creds

	#parse the output for the "received packets" number
	$rxcount = (( $result.ScriptOutput | ? { $_.indexof("Packets") -gt -1 } ).Split(',') | ? { $_.indexof("Received") -gt -1 }).split('=')[1].trim()

	#if we received enough ping replies, consider this a success
	$success = ([int]$rxcount -gt $pingReplyCountThreshold) 

	#however there is one condition where this will be a false positive... gateway reachable but destination not responding
	if ( $result.ScriptOutput | ? { $_.indexof("host unreach") -gt -1 } ) {
		$success = $false
		$rxcount = 0;
	}

	write-host "Full results of ping test..."
	write-host $result.ScriptOutput

	return @($success, $count, $rxcount);
}

function SetGuestIP($ip, $subnet, $gw) {
  $script = @"
	`$iface = (gwmi win32_networkadapter -filter "netconnectionstatus = 2" | select -First 1).interfaceindex
	netsh interface ip set address name=`$iface static $ip $subnet $gw
	netsh interface ipv4 set subinterface `$iface mtu=9000 store=active
"@
  write-host "Script to run: " + $script
  return (Invoke-VMScript -ScriptText $script -VM $vm -GuestCredential $creds)
}

#Tests
# Per Port Group Tests  (Test each port group)

$vmhost = $vm.vmhost
if ($isStandard -eq $false)
{
	foreach($item in $data) {
		if($testPortGroup = $dvs | get-vdportgroup -name $item.PortGroup) {
			($vm | get-Networkadapter)[0] | Set-NetworkAdapter -Portgroup $testPortGroup -confirm:$false
			if( SetGuestIP $item.SourceIP $item.SubnetMask $item.GatewayIP ) {
				echo ("Set Guest IP to " + $item.SourceIP)

				#Run normal ping test
				$pingTestResult = TestPing $item.TestIP $timesToPing $false
				#Add to results
				$thisTest = [ordered]@{"VM" = $vm.name; "TimeStamp" = (Get-Date -f s); "Host" = $vmhost.name;}
				$thisTest["Host"] = $vmhost.name
				$thisTest["PortGroupName"] = $testPortGroup.name
				$thisTest["VlanID"] = $testPortGroup.vlanconfiguration.vlanid
				$thisTest["SourceIP"] = $item.SourceIP
				$thisTest["DestinationIP"] = $item.TestIP
				$thisTest["Result"] = $pingTestResult[0].tostring()
				$thisTest["TxCount"] = $pingTestResult[1].tostring()
				$thisTest["RxCount"] = $pingTestResult[2].tostring()
				$thisTest["JumboFramesTest"] = ""
				$thisTest["Uplink"] = $thisUplink

				$testResults += new-object -typename psobject -Property $thisTest

				#DISABLED JUMBO FRAMES TEST!
				if($false) {
					#Run jumbo frames test
					$pingTestResult = TestPing $item.TestIP $timesToPing $true
					#Add to results
					$thisTest = [ordered]@{"VM" = $vm.name; "TimeStamp" = (Get-Date -f s); "Host" = $vmhost.name;}
					$thisTest["Host"] = $vmhost.name
					$thisTest["PortGroupName"] = $testPortGroup.name
					$thisTest["VlanID"] = $testPortGroup.vlanconfiguration.vlanid
					$thisTest["SourceIP"] = $item.SourceIP
					$thisTest["DestinationIP"] = $item.TestIP
					$thisTest["Result"] = $pingTestResult[0].tostring()
					$thisTest["TxCount"] = $pingTestResult[1].tostring()
					$thisTest["RxCount"] = $pingTestResult[2].tostring()
					$thisTest["JumboFramesTest"] = ""
					$thisTest["Uplink"] = $thisUplink

					$testResults += new-object -typename psobject -Property $thisTest
				}	

			} else {
				$thisTest = [ordered]@{"VM" = $vm.name; "TimeStamp" = (Get-Date -f s); "Host" = $vmhost.name;}
				$thisTest["PortGroupName"] = $testPortGroup.name
				$thisTest["VlanID"] = $testPortGroup.vlanconfiguration.vlanid
				$thisTest["SourceIP"] = $item.SourceIP
				$thisTest["DestinationIP"] = $item.GatewayIP
				$thisTest["Result"] = "false - error setting guest IP"
				$testResults += new-object -typename psobject -Property $thisTest
			}
		} else {
			$thisTest = [ordered]@{"VM" = $vm.name; "TimeStamp" = (Get-Date -f s); "Host" = $vmhost.name;}
			$thisTest["PortGroupName"] = $item.PortGroup
			$thisTest["Result"] = "false - could not find port group"
			$testResults += new-object -typename psobject -Property $thisTest
		}
	}
}
else {
    #This is for a Standard Switch
	foreach($item in $data) {
		$dvs = $vm |get-vmhost | get-virtualswitch -name $dvsName
		if($testPortGroup = $dvs | get-virtualportgroup -name $item.PortGroup) {
			($vm | get-Networkadapter)[0] | Set-NetworkAdapter -Portgroup $testPortGroup -confirm:$false
			if( SetGuestIP $item.SourceIP $item.SubnetMask $item.GatewayIP ) {
				echo ("Set Guest IP to " + $item.SourceIP)

				#Run normal ping test
				$pingTestResult = TestPing $item.TestIP $timesToPing $false
				#Add to results
				$thisTest = [ordered]@{"VM" = $vm.name; "TimeStamp" = (Get-Date -f s); "Host" = $vmhost.name;}
				$thisTest["Host"] = $vmhost.name
				$thisTest["PortGroupName"] = $testPortGroup.name
				$thisTest["VlanID"] = $testPortGroup.vlanid
				$thisTest["SourceIP"] = $item.SourceIP
				$thisTest["DestinationIP"] = $item.TestIP
				$thisTest["Result"] = $pingTestResult[0].tostring()
				$thisTest["TxCount"] = $pingTestResult[1].tostring()
				$thisTest["RxCount"] = $pingTestResult[2].tostring()
				$thisTest["JumboFramesTest"] = ""
				$thisTest["Uplink"] = $thisUplink

				$testResults += new-object -typename psobject -Property $thisTest

				#DISABLED JUMBO FRAMES TEST!
				if($false) {
					#Run jumbo frames test
					$pingTestResult = TestPing $item.TestIP $timesToPing $true
					#Add to results
					$thisTest = [ordered]@{"VM" = $vm.name; "TimeStamp" = (Get-Date -f s); "Host" = $vmhost.name;}
					$thisTest["Host"] = $vmhost.name
					$thisTest["PortGroupName"] = $testPortGroup.name
					$thisTest["VlanID"] = $testPortGroup.vlanid
					$thisTest["SourceIP"] = $item.SourceIP
					$thisTest["DestinationIP"] = $item.TestIP
					$thisTest["Result"] = $pingTestResult[0].tostring()
					$thisTest["TxCount"] = $pingTestResult[1].tostring()
					$thisTest["RxCount"] = $pingTestResult[2].tostring()
					$thisTest["JumboFramesTest"] = ""
					$thisTest["Uplink"] = $thisUplink

					$testResults += new-object -typename psobject -Property $thisTest
				}	

			} else {
				$thisTest = [ordered]@{"VM" = $vm.name; "TimeStamp" = (Get-Date -f s); "Host" = $vmhost.name;}
				$thisTest["PortGroupName"] = $testPortGroup.name
				$thisTest["VlanID"] = $testPortGroup.vlanid
				$thisTest["SourceIP"] = $item.SourceIP
				$thisTest["DestinationIP"] = $item.GatewayIP
				$thisTest["Result"] = "false - error setting guest IP"
				$testResults += new-object -typename psobject -Property $thisTest
			}
		} else {
			$thisTest = [ordered]@{"VM" = $vm.name; "TimeStamp" = (Get-Date -f s); "Host" = $vmhost.name;}
			$thisTest["PortGroupName"] = $item.PortGroup
			$thisTest["Result"] = "false - could not find port group"
			$testResults += new-object -typename psobject -Property $thisTest
		}
	}
}

# Per Host Tests (Test Each Link for Each VLAN ID on each host)
$testPortGroup = $null

if ($isStandard -eq $false)
{

	($testPortGroup = new-vdportgroup $dvs -Name $testPortGroupName -ErrorAction silentlyContinue) -or ($testPortGroup = $dvs | get-vdportgroup -Name $testPortGroupName)
	($vm | get-Networkadapter)[0] | Set-NetworkAdapter -Portgroup $testPortGroup -confirm:$false

	$cluster | get-vmhost | ? {$_.ConnectionState -match "connected" } | foreach {
		$vmhost = $_
		#Migrate VM to new host
		if(Move-VM -VM $vm -Destination $vmhost) {

			foreach($item in $data) {
				#Configure test port group VLAN ID for this particular VLAN test, or clear VLAN ID if none exists
				$myVlanId = $null
				$myVlanId = (get-vdportgroup -name $item.PortGroup).VlanConfiguration.Vlanid
				if($myVlanId) {
					$testPortGroup = $testPortGroup | Set-VDVlanConfiguration -Vlanid $myVlanId
				} else {
					$testPortGroup = $testPortGroup | Set-VDVlanConfiguration -DisableVlan
				}

				if( SetGuestIP $item.SourceIP $item.SubnetMask $item.GatewayIP ) {
					echo ("Set Guest IP to " + $item.SourceIP)

					#Run test on each uplink individually
					$uplinkset = ( ($testPortGroup | Get-VDUplinkTeamingPolicy).ActiveUplinkPort + ($testPortGroup | Get-VDUplinkTeamingPolicy).StandbyUplinkPort ) | sort
					foreach($thisUplink in $uplinkset) {
						#Disable all uplinks from the test portgroup
						$testPortGroup | Get-VDUplinkTeamingPolicy | Set-VDUplinkTeamingPolicy -UnusedUplinkPort $uplinkset
						#Enable  only this uplink for the test portgroup
						$testPortGroup | Get-VDUplinkTeamingPolicy | Set-VDUplinkTeamingPolicy -ActiveUplinkPort $thisUplink

						#Run normal ping test
						$pingTestResult = TestPing $item.TestIP $timesToPing $false
						#Add to results
						$thisTest = [ordered]@{"VM" = $vm.name; "TimeStamp" = (Get-Date -f s); "Host" = $vmhost.name;}
						$thisTest["Host"] = $vmhost.name
						$thisTest["PortGroupName"] = $testPortGroup.name
						$thisTest["VlanID"] = $testPortGroup.vlanconfiguration.vlanid
						$thisTest["SourceIP"] = $item.SourceIP
						$thisTest["DestinationIP"] = $item.TestIP
						$thisTest["Result"] = $pingTestResult[0].tostring()
						$thisTest["TxCount"] = $pingTestResult[1].tostring()
						$thisTest["RxCount"] = $pingTestResult[2].tostring()
						$thisTest["JumboFramesTest"] = ""
						$thisTest["Uplink"] = $thisUplink

						$testResults += new-object -typename psobject -Property $thisTest

						#DISABLED JUMBO FRAMES TEST!
						if($false) {
							#Run jumbo frames test
							$pingTestResult = TestPing $item.TestIP $timesToPing $true
							#Add to results
							$thisTest = [ordered]@{"VM" = $vm.name; "TimeStamp" = (Get-Date -f s); "Host" = $vmhost.name;}
							$thisTest["Host"] = $vmhost.name
							$thisTest["PortGroupName"] = $testPortGroup.name
							$thisTest["VlanID"] = $testPortGroup.vlanconfiguration.vlanid
							$thisTest["SourceIP"] = $item.SourceIP
							$thisTest["DestinationIP"] = $item.TestIP
							$thisTest["Result"] = $pingTestResult[0].tostring()
							$thisTest["TxCount"] = $pingTestResult[1].tostring()
							$thisTest["RxCount"] = $pingTestResult[2].tostring()
							$thisTest["JumboFramesTest"] = ""
							$thisTest["Uplink"] = $thisUplink

							$testResults += new-object -typename psobject -Property $thisTest
						}
					}

					$testPortGroup | Get-VDUplinkTeamingPolicy | Set-VDUplinkTeamingPolicy -ActiveUplinkPort ($uplinkset | sort)

				} else {
					$thisTest = [ordered]@{"VM" = $vm.name; "TimeStamp" = (Get-Date -f s); "Host" = $vmhost.name;}
					$thisTest["PortGroupName"] = $testPortGroup.name
					$thisTest["VlanID"] = $testPortGroup.vlanconfiguration.vlanid
					$thisTest["SourceIP"] = $item.SourceIP
					$thisTest["DestinationIP"] = $item.GatewayIP
					$thisTest["Result"] = "false - error setting guest IP"
					$testResults += new-object -typename psobject -Property $thisTest
				}
			}
		} else {
			$thisTest = [ordered]@{"VM" = $vm.name; "TimeStamp" = (Get-Date -f s); "Host" = $vmhost.name;}
			$thisTest["Result"] = "false - unable to vMotion VM to this host"
			$testResults += new-object -typename psobject -Property $thisTest
		}
	}
}
else {
    #This is for a standard Switch
    $vmhost = $null

    #adding the testPortGroup on all hosts in the cluster
    $cluster | get-vmhost | ? {$_.ConnectionState -match "connected" } | sort | foreach {
		$dvs = get-virtualswitch -Name $dvsName -VMhost $_
    	$dvs | new-virtualportgroup -Name $testPortGroupName -ErrorAction silentlyContinue
    }

    $vmhost = $null
	$cluster | get-vmhost | ? {$_.ConnectionState -match "connected" } | sort | foreach {
		$vmhost = $_
		$dvs = get-virtualswitch -Name $dvsName -VMhost $vmhost
		$testPortGroup = $dvs |get-virtualportgroup -Name $testPortGroupName -VMhost $vmhost -ErrorAction silentlyContinue

		#Migrate VM to new host
		if(Move-VM -VM $vm -Destination $vmhost) {
				write-host -Foregroundcolor:red "Sleeping 5 seconds..."
				start-sleep -seconds 5
				if (($vm | get-Networkadapter)[0] | Set-NetworkAdapter -Portgroup ($dvs |get-virtualportgroup -Name $testPortGroupName -VMhost $vmhost) -confirm:$false -ErrorAction stop)
				{
					write-host -Foregroundcolor:green "Adapter Change successful"
				}else {
				    write-host -Foregroundcolor:red "Cannot change adapter!"
				    #$esxihost = $vm |get-vmhost
				    #$newPortgroup = $esxihost | get-virtualportgroup -Name testPortGroupName -ErrorAction silentlyContinue
				    #if (($vm | get-Networkadapter)[0] | Set-NetworkAdapter -Portgroup ($newPortgroup) -confirm:$false -ErrorAction stop) {
				    #    write-host -Foregroundcolor:green "Adapter Change successful (2nd attempt)"
				    #} else {
				    #    write-host -Foregroundcolor:red "Cannot change Adapter even on 2nd attempt. Exiting script"
				    #    exit 1
				    #}
				}

			foreach($item in $data) {
				#Configure test port group VLAN ID for this particular VLAN test, or clear VLAN ID if none exists
				$myVlanId = $null
				$myVlanId = [int32](get-virtualportgroup -VMhost $vmhost -Standard:$true -name $item.PortGroup).Vlanid
				if($myVlanId) {
					$testPortGroup = $testPortGroup | Set-VirtualPortGroup -Vlanid $myVlanId
				} else {
					$testPortGroup = $testPortGroup | Set-VirtualPortGroup -VlanId 0
				}

				if( SetGuestIP $item.SourceIP $item.SubnetMask $item.GatewayIP ) {
					echo ("Set Guest IP to " + $item.SourceIP)

					#Run test on each uplink individually
					$uplinkset = ( ($testPortGroup | Get-NicTeamingPolicy).ActiveNic + ($testPortGroup |Get-NicTeamingPolicy).StandbyNic ) |sort
					foreach($thisUplink in $uplinkset) {
						#Disable all uplinks from the test portgroup
						$testPortGroup | Get-NicTeamingPolicy | Set-NicTeamingPolicy -MakeNicUnused $uplinkset
						#Enable  only this uplink for the test portgroup
						$testPortGroup | Get-NicTeamingPolicy | Set-NicTeamingPolicy -MakeNicActive $thisUplink

						#Run normal ping test
						$pingTestResult = TestPing $item.TestIP $timesToPing $false
						#Add to results
						$thisTest = [ordered]@{"VM" = $vm.name; "TimeStamp" = (Get-Date -f s); "Host" = $vmhost.name;}
						$thisTest["Host"] = $vmhost.name
						$thisTest["PortGroupName"] = $testPortGroup.name
						$thisTest["VlanID"] = $testPortGroup.vlanid
						$thisTest["SourceIP"] = $item.SourceIP
						$thisTest["DestinationIP"] = $item.TestIP
						$thisTest["Result"] = $pingTestResult[0].tostring()
						$thisTest["TxCount"] = $pingTestResult[1].tostring()
						$thisTest["RxCount"] = $pingTestResult[2].tostring()
						$thisTest["JumboFramesTest"] = ""
						$thisTest["Uplink"] = $thisUplink

						$testResults += new-object -typename psobject -Property $thisTest

						#DISABLED JUMBO FRAMES TEST!
						if($false) {
							#Run jumbo frames test
							$pingTestResult = TestPing $item.TestIP $timesToPing $true
							#Add to results
							$thisTest = [ordered]@{"VM" = $vm.name; "TimeStamp" = (Get-Date -f s); "Host" = $vmhost.name;}
							$thisTest["Host"] = $vmhost.name
							$thisTest["PortGroupName"] = $testPortGroup.name
							$thisTest["VlanID"] = $testPortGroup.vlanid
							$thisTest["SourceIP"] = $item.SourceIP
							$thisTest["DestinationIP"] = $item.TestIP
							$thisTest["Result"] = $pingTestResult[0].tostring()
							$thisTest["TxCount"] = $pingTestResult[1].tostring()
							$thisTest["RxCount"] = $pingTestResult[2].tostring()
							$thisTest["JumboFramesTest"] = ""
							$thisTest["Uplink"] = $thisUplink

							$testResults += new-object -typename psobject -Property $thisTest
						}
					}

					$testPortGroup | Get-NicTeamingPolicy | Set-NicTeamingPolicy -MakeNicActive ($uplinkset | sort)

				} else {
					$thisTest = [ordered]@{"VM" = $vm.name; "TimeStamp" = (Get-Date -f s); "Host" = $vmhost.name;}
					$thisTest["PortGroupName"] = $testPortGroup.name
					$thisTest["VlanID"] = $testPortGroup.vlanid
					$thisTest["SourceIP"] = $item.SourceIP
					$thisTest["DestinationIP"] = $item.GatewayIP
					$thisTest["Result"] = "false - error setting guest IP"
					$testResults += new-object -typename psobject -Property $thisTest
				}
			}
		} else {
			$thisTest = [ordered]@{"VM" = $vm.name; "TimeStamp" = (Get-Date -f s); "Host" = $vmhost.name;}
			$thisTest["Result"] = "false - unable to vMotion VM to this host"
			$testResults += new-object -typename psobject -Property $thisTest
		}
	}
}

#Clean up
if ($isStandard -eq $false)
{
	($vm | get-Networkadapter)[0] | Set-NetworkAdapter -Portgroup (get-vdportgroup $originalVMPortGroup) -confirm:$false
	Remove-VDPortGroup -VDPortGroup $testPortGroup -confirm:$false

} else {
	$tempvm = get-vm $vmName
	$temphost = $tempvm |get-VMhost
	$portGroupToRevertTo = $temphost |get-virtualportgroup -name $temporaryVar -Standard:$true
    ($vm | get-Networkadapter)[0] | Set-NetworkAdapter -Portgroup $portGroupToRevertTo -confirm:$false
    write-host -Foregroundcolor:green "Waiting 5 seconds for $vm to revert back to $temporaryVar"
    start-sleep -seconds 5
    $cluster | get-vmhost | ? {$_.ConnectionState -match "connected" } | foreach {
		$vmhost = $_
		$dvs = $vmhost | get-virtualswitch -Name $dvsName
		$testPortGroup = $dvs | get-virtualportgroup -name $testPortGroupName -Standard:$true -VMhost $vmhost
		remove-virtualportgroup -virtualportgroup $testPortGroup -confirm:$false
	}
}

#Future Test Ideas
#Query driver/firmware for each host's network adapters ?

#Show Results
$testResults | ft
$testResults | Export-CSV -notypeinformation $resultFile
[/code]


## Usage


You can run the script without any parameters if you wish, you will be prompted for each of them.



	
  * clusterName : Name of the cluster in which you want to test your networks

	
  * dvsName : Name of the switch (distributed or standard) you want to test

	
  * isStandard: $true or $false depending on wether your switch is standard ($true) or distributed ($false)

	
  * creds : Prompts for the credentials with which you can run scripts on your test-vm (not vCenter credentials)

	
  * vmName : Name of the windows machine that will be used for testing the networks

	
  * csvFile : path to the map.csv as shown in the requirements

	
  * resultFile : path to where you want the resulting csv to be saved


If you're like me you will pass all arguments when running the script (except maybe creds):


<blockquote>./virtualFrogNetworkTester.ps1 -clusterName VirtualFrog1 -dvsName vSwitch1 -isStandard:$true -vmName networktestingFrogVM -csvFile c:\temp\map.csv -resultFile c:\temp\virtualFrogRocks.csv</blockquote>




## Result


The resulting CSV File will look something like that:

![Screen Shot 2017-07-26 at 14.43.48](https://virtualfrog.files.wordpress.com/2017/07/screen-shot-2017-07-26-at-14-43-48.png)

What does that tell you? Well the script does two tests:



	
  1. It assigns the test-vm to each network from your map.csv, assigns the ip and pings the target

	
  2. Here it goes through all hosts in the cluster, creates a temporary network, assigns the tags from your testnetworks and then goes through the uplinks (setting both unused, then one active, both unused, the other one active).


So after you've run this test you can tell if all of your uplinks have connectivity to all the assigned networks and you can be sure that VMs are able to communicate inside those networks.


## Known Issues


This is something I couldn't figure out. If your VM is assigned to a network with a name like: "vlan-7-192.168.100.0" the script will not be able to reassign this network in the cleaning up part. It sees the value as multiple values and will not set the networkadapter to that network. So just a hint: Do not use dots (.) in your network names!

This probably won't be a problem, but I've only tested this in a vSphere 6.0 U3b Environment with PowerCLI 6.5.1. If you have issues with the script, please let me know over twitter or as a comment.
