---
layout: post
title: Changing default Backup location for MSSQL database
date: 2018-09-21 09:32:35.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- vExpert
- virtualFrog Posts
- Zerto
tags: []
meta:
  _thumbnail_id: '412'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:57:"https://twitter.com/virtual_dd/status/1043040305941962752";}}
  publicize_linkedin_url: www.linkedin.com/updates?topic=6448806000395845632
  timeline_notification: '1537515160'
  _publicize_job_id: '22373906598'
  _publicize_done_9340485: '1'
  _wpas_done_9383931: '1'
  publicize_twitter_user: virtual_dd
  _publicize_done_14862392: '1'
  _wpas_done_14749148: '1'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2018/09/21/changing-default-backup-location-for-mssql-database/"
---
Just yesterday I upgraded Zerto from 6.0 U2 to 6.5 at a customer site. The customer had several different Zerto Virtual Manager (ZVM) instances and one of which was isolated from the rest of the network in a secured zone.

The upgrade process failed right at the time when the upgrade process tried to backup the Zerto SQL database:

![error]({{ site.baseurl }}/assets/images/error.jpg)

> An error has occurred.  
> Package : Zerto Virtual Replication Error : 1: DatabaseCustomActions.ZvmBackupDatabase 2: Cannot open backup device 'BACKUP DEVICE\zvm\_storage\_TIMESTAMP.bak'. Operating system error 53(The network path was not found.).  
> BACKUP DATABASE is terminating abnormally.

<!--more-->At first I figured this must be a setting in Zerto. However I found no such setting and turned my attention to the SQL server. After some research I found the culprit: The default backup location was set to the company's default share for SQL backups. This share however was not accessible from that isolated environment. The solution was to reconfigure the default backup location on the sql server and after that the upgrade succeeded.

Here are the steps to change the default backup setting in a MS SQL instance:

1. Start SQL Server Management Studio (SSMS) and connect to your database server

![manager]({{ site.baseurl }}/assets/images/manager.png)

2. Open the properties of the server instance (not the database itself):

![properties]({{ site.baseurl }}/assets/images/properties.png)

3. Choose the "Database Settings" and change the default location for Backup:

![backupsettings]({{ site.baseurl }}/assets/images/backupsettings.png)

I hope this helps someone down the road.

