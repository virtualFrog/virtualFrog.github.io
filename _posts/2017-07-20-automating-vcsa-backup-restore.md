---
layout: post
title: Automating VCSA Backup & Restore
date: 2017-07-20 10:22:26.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Scripting
- virtualFrog Posts
tags:
- VMware
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _thumbnail_id: '235'
  _publicize_job_id: '7283400606'
  _publicize_done_external: a:1:{s:7:"twitter";a:1:{i:9383931;s:56:"https://twitter.com/virtual_dd/status/887950821257879552";}}
  _publicize_done_9340485: '1'
  _wpas_done_9383931: '1'
  publicize_twitter_user: virtual_dd
  publicize_linkedin_url: https://www.linkedin.com/updates?discuss=&scope=391645417&stype=M&topic=6293716514415611904&type=U&a=v4Xp
  _publicize_done_14862392: '1'
  _wpas_done_14749148: '1'
author:
  login: virtualfrog
  email: dario.doerflinger@gmail.com
  display_name: virtualFrog
  first_name: ''
  last_name: ''
permalink: "/2017/07/20/automating-vcsa-backup-restore/"
---
<p>I've recently stumbled upon a VMware communities thread where someone had trouble backing up the VCSA database. The were using someone else's script, so I thought I'd share the wrapper script I wrote around two python scripts that VMware has published in its <a href="https://kb.vmware.com/kb/2091961" target="_blank" rel="noopener">KB 2091961</a></p>
<p><!--more--></p>
<p>Update: this script is now published at <a href="https://github.com/virtualFrog/PowerCLI-Scripts" target="_blank" rel="noopener">GitHub</a>.</p>
<h2>The script</h2>
<p>[code language="bash"]<br />
#!/bin/bash</p>
<p>#***************************************************************<br />
# Get Options from the command line<br />
#***************************************************************<br />
while getopts "b:r:h" options<br />
do<br />
case $options in<br />
                b ) opt_b=$OPTARG;;<br />
                r ) opt_r=$OPTARG;;<br />
                h ) opt_h=1;;<br />
                \? ) opt_h=1;;<br />
esac<br />
done</p>
<p>###############################################<br />
# bkp_rst.sh<br />
#=============================================<br />
# Name: bkp_rst.sh<br />
# Datum: 24.07.2017<br />
# Ziel: Mache ein Backup der vCenter Appliance Datenbank, Restore die Datenbank<br />
#<br />
# Autor: Dario aka virtualFrog<br />
version="1.5"<br />
#<br />
# Changelog:<br />
# 0.1                   dario           Initial-Setup<br />
# 1.0                   dario           Testing, Testing, Testing<br />
# 1.1                   dario           Added dynamic filename &amp; cleanUp Function<br />
# 1.2                   dario           Added local cleanup<br />
# 1.3                   dario           Changed local cleanup to delete all but most recent 3 files<br />
# 1.4                   dario           Added checkForErrors function and writeMail function<br />
# 1.5                   dario           Added CheckForErrors before every exit statement</p>
<p># Variablen<br />
# ---------</p>
<p>#log-Variablen<br />
log=/var/log/vmware_backup_restore.log</p>
<p>#Mail variabeln<br />
recipients="test@gmail.com,test3@gmail.com"</p>
<p>#NFS-Parameter<br />
nfsmount="vcsa"<br />
nfsserver=""<br />
nfsoptions="nfs4"</p>
<p># Funktionen<br />
# ----------</p>
<p>function print_banner<br />
{<br />
        echo "</p>
<p>___  __ ____   ___________<br />
\  \/ // ___\ /  ___/\__  \</p>
<p> \   /\  \___ \___ \  / __ \_<br />
  \_/  \___  &gt;____  &gt;(____  /<br />
           \/     \/      \/<br />
                            "<br />
echo "v$version"<br />
}</p>
<p>function get_parameter<br />
{<br />
        echo "GET_PARAMETER"</p>
<p>        if [ $opt_h ]; then print_help<br />
        fi</p>
<p>        #--------------------------------------<br />
        # Check to see if -b was passed in<br />
        #--------------------------------------
  
 if [$opt\_b]; then  
 backup\_string=$opt\_b  
 echo -e "\tBackup: $backup\_string"  
 echo "Backup: $backup\_string" \>\> $log  
 echo ""  
 validate\_backup\_path  
 filename=`hostname`-`date +%Y-%d-%m:%H:%M:%S`.bak  
 todo="backup"  
 echo ""  
 elif [$opt\_r]; then  
 restore\_string=$opt\_r  
 echo -e "\tRestore: $restore\_string"  
 echo "Restore: $restore\_string" \>\> $log  
 echo ""  
 validate\_restore\_path  
 todo="restore"  
 echo ""  
 else  
 echo "warning: no parameters supplied." \>\> $log  
 echo -e "\tKeine Parameter gefunden."  
 print\_help  
 fi

}

function print\_help  
{  
 echo "Gueltige Parameter: -b -r "  
 echo ""  
 echo "To restore you should mount the volume ($nfsmount) on ($nfsserver) which this script copies the backups to"  
 exit 0  
}

function init\_log  
{

echo `date` \> $log  
 echo "fzag\_bkp\_rst.sh in der Version $version auf `hostname -f`" \>\> $log  
 echo "\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_" \>\> $log

#echo "Nun startet das Script in der Version $version"  
 echo ""  
}  
function validate\_backup\_path  
{  
 echo "VALIDATE BACKUP PATH"  
 echo -e "\tChecking Backup Path: $backup\_string"  
 if [! -d "$backup\_string"]; then  
 echo -e "\tDirectory did not exist. Creating.."  
 mkdir -p $backup\_string  
 echo -e "\tDirectory created."  
 else  
 echo -e "\tDirectory does exist."  
 fi  
 echo ""  
}

function validate\_restore\_path  
{  
 echo "VALIDATE RESTORE PATH"  
 echo -e "\tChecking Backup Path: $restore\_string"  
 if [-f $restore\_string];  
 then  
 echo -e "\tFile $restore\_string exists."  
 echo "File $restore\_string exists." \>\> $log  
 else  
 echo -e "\tFile $restore\_string does not exist. Exiting"  
 echo "error: $restore\_string does not exist." \>\> $log  
 checkForErrors  
 exit 1  
 fi  
 echo ""  
}  
function check\_python\_scripts  
{  
 echo "CHECK PYTHON SCRIPTS"  
 echo -e "\tChecking if Python Scripts are available"  
 DIR="$( cd "$( dirname "${BASH\_SOURCE[0]}" )" && pwd )"  
 if [-f "$DIR/backup\_lin.py"]; then  
 echo -e "\tBackup Script found, continue.."  
 echo "script for backup found" \>\> $log  
 if [-f "$DIR/restore\_lin.py"]; then  
 echo -e "\tRestore Script found, continue.."  
 echo "script for restore found" \>\> $log  
 else  
 echo -e "\tNo restore script found, exit"  
 echo "error: no python restore script found" \>\> $log  
 checkForErrors  
 exit 1  
 fi  
 else  
 echo -e "\tNo backup script found, exit"  
 echo "error: no python backup scripts found" \>\> $log  
 checkForErrors  
 exit 1  
 fi  
 echo ""  
}

function create\_backup  
{  
 echo "CREATE BACKUP"  
 echo -e "\tCreating the Backup in the file $backup\_string/bk.bak:"  
 DIR="$( cd "$( dirname "${BASH\_SOURCE[0]}" )" && pwd )"

echo `python $DIR/backup_lin.py -f $backup_string/$filename` \>\> $log  
 if [$? -ne 0]; then  
 echo -e "\tError: Return code from backup job was not 0"  
 echo "error: return code from backup job was not 0" \>\> $log  
 checkForErrors  
 exit 1  
 else  
 if [-f "$backup\_string/$filename"]; then  
 echo -e "\tSuccess! Backup created."  
 echo "success: backup created" \>\> $log  
 fi  
 fi  
 echo ""  
}

function restore\_db  
{  
 echo "RESTORE DB"  
 echo -e "\tRestoring Database with supplied Backup-File: $restore\_string"  
 DIR="$( cd "$( dirname "${BASH\_SOURCE[0]}" )" && pwd )"  
 echo `python $DIR/restore_lin.py -f $restore_string` \>\> $log  
 if [$? -ne 0]; then  
 echo -e "\tError: Return code from restore job was not 0"  
 echo "error: return code from restore job was not 0" \>\> $log  
 fi  
 echo ""

}

function mountNFS  
{  
 echo "MOUNT NFS"  
 echo -e "\tMounting the given NFS Volume ($nfsmount) on server ($nfsserver)"  
 if [[! -d /mnt/bkp]]; then  
 mkdir -p /mnt/bkp  
 fi

mount -t $nfsoptions $nfsserver:$nfsmount /mnt/bkp  
 if [[$? -ne 0]]; then  
 echo -e "\tMount was not successful. Abort the operation"  
 echo "mount was not successful" \>\> $log  
 checkForErrors  
 exit 1  
 fi  
 echo ""  
 return 0  
}

function copyBackup  
{  
 echo "COPY BACKUP"  
 echo -e "\tCopying Backup to NFS Mount"  
 cp $backup\_string/$filename /mnt/bkp/  
 if [[$? -ne 0]]; then  
 echo -e "\tCopy Operation was not successful. Abort the operation"  
 echo "copy was not successful" \>\> $log  
 checkForErrors  
 exit 1  
 fi  
 echo ""  
}

function unMountNFS  
{  
 echo "UMOUNT NFS"  
 echo -e "\tUnmounting the NFS Mount"  
 umount /mnt/bkp  
 if [[$? -ne 0]]; then  
 echo -e "\tumount Operation was not successful."  
 echo "umount was not successful" \>\> $log  
 fi  
 echo ""  
}

function cleanUp  
{  
 echo "CLEAN UP"  
 echo -e "\tCleaning up Backups older than 4 days and all local files but the 3 most recent"

find /mnt/bkp -mtime +4 -exec rm -rf {} \;  
 if [[$? -ne 0]]; then  
 echo -e "\tNFS cleanUp Operation was not successful."  
 echo "NFS cleanUp was not successful" \>\> $log  
 fi

cd $backup\_string  
 ls -t1 $backup\_string |tail -n +3 | xargs rm  
 if [[$? -ne 0]]; then  
 echo -e "\tLocal cleanUp Operation was not successful."  
 echo "local cleanUp was not successful" \>\> $log  
 fi  
 echo ""  
}

function writeEmail  
{  
 echo "WRITE EMAIL"  
 echo -e "\tSending the logfile as a mail to $recipients"  
 subject="VCSA Backup has run into a error"  
 from="root@`hostname -f`"  
 body=`cat $log`

/usr/sbin/sendmail "$recipients" \<\> $log  
 service vmware-vdcs stop \>\> $log  
 restore\_db  
 service-control --start vmware-vpxd \>\> $log  
 service-control --start vmware-vdcs \>\> $log  
 unMountNFS  
 checkForErrors  
fi  
[/code]

## Requirements

The script expects the python scripts from the [KB 2091961](https://kb.vmware.com/kb/2091961) to be in the same directory as the scripts. It will stop if they are not there.

## Usage

Even though I went the extra mile to provide a syntax helper (so if you run the script with no parameters it will display which parameters you should use) I will tell you here exactly how to use the script.

First thing to do is to fill in the variables for the nfs server and mount where you want to store your backup:

[code language="bash"]  
#NFS-Parameter  
nfsmount="vcsa"  
nfsserver="192.168.1.1"  
nfsoptions="nfs4"  
[/code]

### Backup

[code language="bash"]  
./bkp\_rst\_1.2.sh -b /tmp/backup  
[/code]

This will create the directory if it doesn't exist and will keep some backups local. There is a cleanup function that deletes stuff older than 4 days.

### Restore

[code language="bash"]  
./bkp\_rst\_1.2.sh -r /tmp/backup/bk.bak  
[/code]

In order to restore you need to specify a file to restore from. If it is one on the nfs server running the script without parameters will tell you the nfs server and which mount you've defined in the script.

### Crontab

This script is made to run as a cronjob. So just use "crontab -e" (edit mode) and add this line:

[code language="bash"]  
0 \* \* \* \* /bin/bash /usr/bin/bkp\_rst\_1.2.sh -b /tmp/backup  
[/code]

**Attention for vSphere 6.5:**  
There is a bug in VCSA 6.5 GA which prevents cron to run scripts.  
You'll need to change the file "/etc/pam.d/crond". Change the 3 references to "password-auth" to "system-auth" and it will work. ~~I hope this will get fixed in Update 1.~~

Update: It has not been fixed in vSphere 6.5 Update 1 GA, we might see a fix coming within a patch.

If you need help with this reach out to me on twitter or hit me up with a comment.

Here is a sample output of the script when it is run manually:  
 ![Screen Shot 2017-07-20 at 10.18.06]({{ site.baseurl }}/assets/images/screen-shot-2017-07-20-at-10-18-06.png)

And here is the output if run without parameters:  
 ![Screen Shot 2017-07-20 at 10.18.24]({{ site.baseurl }}/assets/images/screen-shot-2017-07-20-at-10-18-24.png)

## Update

I've just updated the script to version 1.5. It now includes a better local cleanup (keeps only the 3 most recent files) has a errorchecking and will write a e-mail if there was a error in the log.

