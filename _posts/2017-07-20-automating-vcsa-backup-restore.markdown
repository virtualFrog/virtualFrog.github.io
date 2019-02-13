---
author: virtualfrog
comments: true
date: 2017-07-20 08:22:26+00:00
layout: post
link: https://virtualfrog.wordpress.com/2017/07/20/automating-vcsa-backup-restore/
slug: automating-vcsa-backup-restore
title: Automating VCSA Backup & Restore
wordpress_id: 197
categories:
- General
- Scripting
tags:
- VMware
---

I've recently stumbled upon a VMware communities thread where someone had trouble backing up the VCSA database. The were using someone else's script, so I thought I'd share the wrapper script I wrote around two python scripts that VMware has published in its [KB 2091961](https://kb.vmware.com/kb/2091961)

<!-- more -->

Update: this script is now published at [GitHub](https://github.com/virtualFrog/PowerCLI-Scripts).


## The script


[code language="bash"]
#!/bin/bash

#***************************************************************
# Get Options from the command line
#***************************************************************
while getopts "b:r:h" options
do
case $options in
                b ) opt_b=$OPTARG;;
                r ) opt_r=$OPTARG;;
                h ) opt_h=1;;
                \? ) opt_h=1;;
esac
done

###############################################
# bkp_rst.sh
#=============================================
# Name: bkp_rst.sh
# Datum: 24.07.2017
# Ziel: Mache ein Backup der vCenter Appliance Datenbank, Restore die Datenbank
#
# Autor: Dario aka virtualFrog
version="1.5"
#
# Changelog:
# 0.1                   dario           Initial-Setup
# 1.0                   dario           Testing, Testing, Testing
# 1.1                   dario           Added dynamic filename & cleanUp Function
# 1.2                   dario           Added local cleanup
# 1.3                   dario           Changed local cleanup to delete all but most recent 3 files
# 1.4                   dario           Added checkForErrors function and writeMail function
# 1.5                   dario           Added CheckForErrors before every exit statement

# Variablen
# ---------

#log-Variablen
log=/var/log/vmware_backup_restore.log

#Mail variabeln
recipients="test@gmail.com,test3@gmail.com"

#NFS-Parameter
nfsmount="vcsa"
nfsserver=""
nfsoptions="nfs4"

# Funktionen
# ----------

function print_banner
{
        echo "

___  __ ____   ___________
\  \/ // ___\ /  ___/\__  \

 \   /\  \___ \___ \  / __ \_
  \_/  \___  >____  >(____  /
           \/     \/      \/
                            "
echo "v$version"
}

function get_parameter
{
        echo "GET_PARAMETER"

        if [ $opt_h ]; then print_help
        fi

        #--------------------------------------
        # Check to see if -b was passed in
        #--------------------------------------
        if [ $opt_b ]; then
                backup_string=$opt_b
                echo -e "\tBackup: $backup_string"
                echo "Backup: $backup_string" >> $log
                echo ""
                validate_backup_path
                filename=`hostname`-`date +%Y-%d-%m:%H:%M:%S`.bak
                todo="backup"
                echo ""
        elif [ $opt_r ]; then
                restore_string=$opt_r
                echo -e "\tRestore: $restore_string"
                echo "Restore: $restore_string" >> $log
                echo ""
                validate_restore_path
                todo="restore"
                echo ""
        else
                echo "warning: no parameters supplied." >> $log
                echo -e "\tKeine Parameter gefunden."
                print_help
        fi

}

function print_help
{
        echo "Gueltige Parameter: -b  -r "
        echo ""
        echo "To restore you should mount the volume ($nfsmount) on ($nfsserver) which this script copies the backups to"
        exit 0
}

function init_log
{

                echo `date` > $log
                echo "fzag_bkp_rst.sh in der Version $version auf `hostname -f`" >> $log
                echo "____________________________________________" >> $log

                #echo "Nun startet das Script in der Version $version"
                echo ""
}
function validate_backup_path
{
        echo "VALIDATE BACKUP PATH"
        echo -e "\tChecking Backup Path: $backup_string"
        if [ ! -d "$backup_string" ]; then
                echo -e "\tDirectory did not exist. Creating.."
                mkdir -p $backup_string
                echo -e "\tDirectory created."
        else
                echo -e "\tDirectory does exist."
        fi
        echo ""
}

function validate_restore_path
{
        echo "VALIDATE RESTORE PATH"
        echo -e "\tChecking Backup Path: $restore_string"
        if [ -f $restore_string ];
        then
                echo -e "\tFile $restore_string exists."
                echo "File $restore_string exists." >> $log
        else
                echo -e "\tFile $restore_string does not exist. Exiting"
                echo "error: $restore_string does not exist." >> $log
                checkForErrors
                exit 1
        fi
        echo ""
}
function check_python_scripts
{
        echo "CHECK PYTHON SCRIPTS"
        echo -e "\tChecking if Python Scripts are available"
        DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
        if [ -f "$DIR/backup_lin.py" ]; then
                echo -e "\tBackup Script found, continue.."
                echo "script for backup found" >> $log
                if [ -f "$DIR/restore_lin.py" ]; then
                        echo -e "\tRestore Script found, continue.."
                        echo "script for restore found" >> $log
                else
                        echo -e "\tNo restore script found, exit"
                        echo "error: no python restore script found" >> $log
                        checkForErrors
                        exit 1
                fi
        else
                echo -e "\tNo backup script found, exit"
                echo "error: no python backup scripts found" >> $log
                checkForErrors
                exit 1
        fi
        echo ""
}

function create_backup
{
        echo "CREATE BACKUP"
        echo -e "\tCreating the Backup in the file $backup_string/bk.bak:"
        DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"

        echo `python $DIR/backup_lin.py -f $backup_string/$filename` >> $log
        if [ $? -ne 0 ]; then
                echo -e "\tError: Return code from backup job was not 0"
                echo "error: return code from backup job was not 0" >> $log
                checkForErrors
                exit 1
        else
                if [ -f "$backup_string/$filename" ]; then
                        echo -e "\tSuccess! Backup created."
                        echo "success: backup created" >> $log
                fi
        fi
        echo ""
}

function restore_db
{
        echo "RESTORE DB"
        echo -e "\tRestoring Database with supplied Backup-File: $restore_string"
        DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
        echo `python $DIR/restore_lin.py -f $restore_string` >> $log
        if [ $? -ne 0 ]; then
                echo -e "\tError: Return code from restore job was not 0"
                echo "error: return code from restore job was not 0" >> $log
        fi
        echo ""

}

function mountNFS
{
        echo "MOUNT NFS"
        echo -e "\tMounting the given NFS Volume ($nfsmount) on server ($nfsserver)"
        if [[ ! -d /mnt/bkp ]]; then
                mkdir -p /mnt/bkp
        fi

        mount -t $nfsoptions $nfsserver:$nfsmount /mnt/bkp
        if [[ $? -ne 0 ]]; then
                echo -e "\tMount was not successful. Abort the operation"
                echo "mount was not successful" >> $log
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
        cp $backup_string/$filename /mnt/bkp/
        if [[ $? -ne 0 ]]; then
                echo -e "\tCopy Operation was not successful. Abort the operation"
                echo "copy was not successful" >> $log
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
        if [[ $? -ne 0 ]]; then
                echo -e "\tumount Operation was not successful."
                echo "umount was not successful" >> $log
        fi
        echo ""
}

function cleanUp
{
        echo "CLEAN UP"
        echo -e "\tCleaning up Backups older than 4 days and all local files but the 3 most recent"

        find /mnt/bkp -mtime +4 -exec rm -rf {} \;
        if [[ $? -ne 0 ]]; then
                echo -e "\tNFS cleanUp Operation was not successful."
                echo "NFS cleanUp was not successful" >> $log
        fi

        cd $backup_string
        ls -t1 $backup_string |tail -n +3 | xargs rm
        if [[ $? -ne 0 ]]; then
                echo -e "\tLocal cleanUp Operation was not successful."
                echo "local cleanUp was not successful" >> $log
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

        /usr/sbin/sendmail "$recipients" <> $log
        service vmware-vdcs stop >> $log
        restore_db
        service-control --start vmware-vpxd >> $log
        service-control --start vmware-vdcs >> $log
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
./bkp_rst_1.2.sh -b /tmp/backup
[/code]

This will create the directory if it doesn't exist and will keep some backups local. There is a cleanup function that deletes stuff older than 4 days.


### Restore


[code language="bash"]
./bkp_rst_1.2.sh -r /tmp/backup/bk.bak
[/code]

In order to restore you need to specify a file to restore from. If it is one on the nfs server running the script without parameters will tell you the nfs server and which mount you've defined in the script.


### Crontab


This script is made to run as a cronjob. So just use "crontab -e" (edit mode) and add this line:

[code language="bash"]
0 * * * * /bin/bash /usr/bin/bkp_rst_1.2.sh -b /tmp/backup
[/code]

**Attention for vSphere 6.5:**
There is a bug in VCSA 6.5 GA which prevents cron to run scripts.
You'll need to change the file "/etc/pam.d/crond". Change the 3 references to "password-auth" to "system-auth" and it will work. <del>I hope this will get fixed in Update 1.</del>

Update: It has not been fixed in vSphere 6.5 Update 1 GA, we might see a fix coming within a patch.

If you need help with this reach out to me on twitter or hit me up with a comment.

Here is a sample output of the script when it is run manually:
![Screen Shot 2017-07-20 at 10.18.06](https://virtualfrog.files.wordpress.com/2017/07/screen-shot-2017-07-20-at-10-18-06.png)

And here is the output if run without parameters:
![Screen Shot 2017-07-20 at 10.18.24](https://virtualfrog.files.wordpress.com/2017/07/screen-shot-2017-07-20-at-10-18-24.png)


## Update


I've just updated the script to version 1.5. It now includes a better local cleanup (keeps only the 3 most recent files) has a errorchecking and will write a e-mail if there was a error in the log.
