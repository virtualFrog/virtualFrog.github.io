---
layout: post
title: 'PowerCLI: Assigning Tags based on VM Notes'
date: 2019-02-21 08:54:53.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- PowerCLI
- Scripting
- vExpert
- virtualFrog Posts
- VMware
tags:
- and
- at
- create
- here
- if
- just
meta:
  _post_tag_term_meta: and
  _thumbnail_id: '29'
  _rest_api_published: '1'
  timeline_notification: '1550735705'
  _rest_api_client_id: "-1"
  _publicize_job_id: '27868172637'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:57:"https://twitter.com/virtual_dd/status/1098491294693896192";}}
  _publicize_done_9340485: '1'
  _wpas_done_9383931: '1'
  publicize_twitter_user: virtual_dd
  publicize_linkedin_url: ''
  _publicize_done_21519738: '1'
  _wpas_done_22130612: '1'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2019/02/21/powercli-assigning-tags-based-on-vm-notes/"
excerpt: Addressing the problem with not finding VMs based on information in the notes
  when using the HTML5 client.
---
This week's challenge was introduced to me at a customer engagement. I helped the customer to migrate from vSphere 6.5 to the latest vSphere 6.7 Update 1 build. The customer was at first very impressed with the (finally) completed HTML5 client.

### The problem with the new HTML5 client: finding VMs is hard

As the customer started working with the new HTML5 client he quickly found out that the search in the HTML5 client is not as good as the search in the flex client. **He was not able to find his VMs based on the information he had added to each VM in the "Notes" property**. I immediately thought: PowerCLI can surely help out here.<!--more-->I told him that the fastest way to find information on VMs is to work with Tags. I also told him about all the newer products that use tags like the automated response in NSX in conjunction with 3rd party antivirus solutions. He was quickly onboard with using tags. I think the "It's just like a post-it" got through to him.

### Searching custom attributes is useless

The starting ground was pretty easy. The customer not only used notes but had also defined a couple of custom attributes. Those were still searchable in the new client but you had to click way too much and be much too specific in the search mask to achieve this:

![Screenshot 2019-02-21 at 08.22.53]({{ site.baseurl }}/assets/images/screenshot-2019-02-21-at-08.22.53.png)

One has to specify exactly which attribute and what values it should have in order for vCenter to find anything. To me that is total nonsense and therefore useless. The customer agreed. So we decided to migrate all the custom attributes to tags as well. As this was a one-time procedure I created the following logic for the customer:

[code language="powershell"]  
#just to be thourough here is the code to actually create the Tag-Category  
New-TagCategory -Name "Install Date" -Cardinality "Single" -EntityType "VirtualMachine" -Confirm:$false  
foreach ($currentVm in get-vm) {  
 #here we extract the value from our custom attribute  
 $installDate = $currentVm.CustomFields | Where-Object {$\_.Key -match "Install"} | Select-Object -ExpandProperty Value  
 if ($installDate -eq "")  
{  
 #if the VM did not have the custom value defined we go ahead and give a value  
 $installDate = "01.01.1900"  
 #and we add the custome attribute as well  
 $currentVm | Set-Annotation -CustomAttribute "Install Date" -Value $installDate -WhatIf  
 }  
 try  
{  
 #if the tag is not found we execute the "catch"  
 get-tag -Name $installDate -ErrorAction Stop  
 }  
 catch  
{  
 #create the tag if it does not exist  
 New-Tag -Name $installDate -Category "Install Date" -WhatIf -Confirm:$false  
 }  
 finally  
{  
 #at this point we know the tag exists so we assign it to the current VM from the loop  
 New-TagAssignment -Tag (Get-Tag -Name $installDate) -Entity $currentVm -WhatIf  
 }

}  
[/code]

I changed the static parts and ran this little script for each of the customer's custom attributes. Afterwards the customer decided to remove the custom attributes all-together and use the tags from now on.

### Getting information from notes to tags

The next challenge was the notes property of the VMs. The customer wrote all kinds of information in that field and used this information to quickly find VMs in the day to day operations. How can we enable the customer to find his VMs again? It's pretty easy we create a new tag category for "Notes" and loop through all the VMs and convert their notes to a tag which we then assign to that VM. The logic is basically the same as in the above example.

However, the customer raised a good point during the discussion:

> I love using the notes property of the VMs because that is actually written to the VMX file and therefore also recovered when I have to restore a VM from backups.

This actually got me thinking, is there a backup vendor that can handle tag-assignment and re-assign those tags after a complete VM restore? I do not think so but did not look too much into this.

> Update 1:  
> After publishing this blog a couple of colleagues from the VMware vExpert community reached out to me regarding this. It seems that Veeam is able to restore tags when a VM is restored. Good to know.  
> I hope more backup vendors will support restoring tags (and maybe custom attributes) in the future.

After discussing this we decided that the customer keeps using the VM notes property for his operational tasks and I will write a script for him that will automatically take the notes value and create a tag from it. A key thing the customer took away from that is that it makes sense to standardize the format of his VM notes. I told him that whenever he has done this and wants to remove all tags with the old VM notes syntax he should just remove the tag category (which will remove all tags as well). The next time the script runs all tags are created again.

The resulting script is too large to put into this post, but as always I have put it on my [GitHub account here](https://github.com/virtualFrog/PowerCLI-Scripts/blob/master/Set-VMTagFromNotes.ps1).

The script will write a logfile where you can track which VM has gotten assigned which tag. We have decided on a 4-hour schedule to run the script. Each 4 hours all the information from the notes are taken and used as tags and the information is instantly searchable in the HTML5 client.

Feel free to use the script and please give me any feedback. On thing that could probably improve the script tremendously is to handle multi-line notes and split them into different tags, but that is a challenge for another day.

_The script has a requirement for the [PSLogging](https://www.powershellgallery.com/packages/PSLogging/) Module from the [Powershellgallery](https://www.powershellgallery.com)._

