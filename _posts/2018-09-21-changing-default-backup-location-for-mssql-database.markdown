---
author: virtualfrog
comments: true
date: 2018-09-21 07:32:35+00:00
layout: post
link: https://virtualfrog.wordpress.com/2018/09/21/changing-default-backup-location-for-mssql-database/
slug: changing-default-backup-location-for-mssql-database
title: Changing default Backup location for MSSQL database
wordpress_id: 962
categories:
- General
- vExpert
- Zerto
---

Just yesterday I upgraded Zerto from 6.0 U2 to 6.5 at a customer site. The customer had several different Zerto Virtual Manager (ZVM) instances and one of which was isolated from the rest of the network in a secured zone.

The upgrade process failed right at the time when the upgrade process tried to backup the Zerto SQL database:

![error](https://virtualfrog.files.wordpress.com/2018/09/error.jpg)


<blockquote>An error has occurred.
Package : Zerto Virtual Replication Error : 1: DatabaseCustomActions.ZvmBackupDatabase 2: Cannot open backup device 'BACKUP DEVICE\zvm_storage_TIMESTAMP.bak'. Operating system error 53(The network path was not found.).
BACKUP DATABASE is terminating abnormally.</blockquote>


<!-- more -->At first I figured this must be a setting in Zerto. However I found no such setting and turned my attention to the SQL server. After some research I found the culprit: The default backup location was set to the company's default share for SQL backups. This share however was not accessible from that isolated environment. The solution was to reconfigure the default backup location on the sql server and after that the upgrade succeeded.

Here are the steps to change the default backup setting in a MS SQL instance:

1. Start SQL Server Management Studio (SSMS) and connect to your database server

![manager](https://virtualfrog.files.wordpress.com/2018/09/manager.png)

2. Open the properties of the server instance (not the database itself):

![properties](https://virtualfrog.files.wordpress.com/2018/09/properties.png)

3. Choose the "Database Settings" and change the default location for Backup:

![backupsettings](https://virtualfrog.files.wordpress.com/2018/09/backupsettings.png)

I hope this helps someone down the road.
