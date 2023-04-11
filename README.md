# Ubuntu-autoupdate
Automatic Package Updates
## 1. update your system
```
apt-get update
```
## 2. install Cron-apt
```
sudo apt-get install cron-apt
```
By default, cron-apt will execute automatically at 4am.

If your system is normally turned off at 4am, you can either edit /etc/cron.d/cron-apt and change the schedule to a time when your PC is likely to be switched on, or you can use Anacron to make it run when your PC is next turned on.

Making cron-apt execute regularly with Anacron
Anacron makes sure that all the tasks listed in /etc/cron.hourly , /etc/cron.daily , /etc/cron.weekly and /etc/cron.monthly are performed, even if the PC is switched off when they'd normally happen. Overdue tasks are run when the computer is next turned on. On the good side, it never misses a task. On the bad side, if the tasks take a lot of time, your PC might seem a little slow when you first turn it on after a long time being turned off.

Optional: Comment out cron-apt

Anacron does not look in /etc/cron.d . You may wish to comment out the line in /etc/cron.d/cron-apt which would otherwise cause it to run at 4am. It won't do any harm to leave it there (you can run cron-apt many times a day without problems), but why would you need to run it both in cron.d and cron.daily or cron.weekly? For the sake of tidyness you can prefix the line:
```
0 4  * * * root test -x /usr/sbin/cron-apt && /usr-sbin/cron-apt
```
...with a # , which will prevent it from executing at 4am:
```
# 0 4  * * * root test -x /usr/sbin/cron-apt && /usr-sbin/cron-apt
```
Add cron-apt to cron.daily

To make cron-apt be executed by Anacron, create a link in /etc/cron.daily or /etc/cron.weekly. For example:
```
sudo ln -s /usr/sbin/cron-apt /etc/cron.weekly/
```
Manually apply the updates

Cron-apt only downloads new packages. If you are a desktop user, you will be notified that there are updates waiting by a down-arrow icon in the notification area (it will not appear until there are updates ready). Click this and it will prompt you through the update process.

Alternatively, from the command line you can do:
```
sudo apt-get dist-upgrade
```
Advanced Alternatives
Below are alternatives to using cron-apt, for advanced users only. You should use them instead of cron-apt, but there is no point doing them in addition cron-apt. You can remove cron-apt with:
```
sudo apt-get remove cron-apt
```
Advanced Alternative: Automatic Weekly Package Updates Using Cron And Apt-Get
WARNING: As with any system changes, the potential for creating new or additional problems may occur. Please be sure to backup your data and configurations! Use this document at your own risk.

Keeping your Ubuntu (or Debian based) system packages regularly updated not only helps to better secure your system, but also helps keep it running bug free. This howto discusses how to setup a very easy and simple Cron job on your system that will update your system packages weekly, and remove any old unused packages no longer installed after updating.

(This document assumes an always or mostly always on system with a broadband, not dial-up, Internet connection.)

Creating the Weekly Cron Job File
First you will need to create the cron job file. You can use a simple text editor to create the file and save it in your home directory. In Ubuntu, open Applications > Accessories > Text Editor. In the Text Editor, type the following lines:
```
 #!/bin/bash
apt-get update
apt-get upgrade -y
apt-get autoclean
```
Now click Save and name the file something like "autoupdt". The default directory should be your home directory, but verify to be certain. (The steps following this will assume the file has been saved in your home directory.)

Moving the Cron Job File to Cron.Weekly
Now that you have created the cron job file, it needs to be moved into the weekly cron directory so that it will be run automatically on a weekly basis. To do this, we first need to open a command line terminal. In Ubuntu, click Applications > System Tools > Terminal. Now you should see your terminal prompt. At the prompt type "ls" and press Enter (or Return on some keyboards). In the list you should see your newly created file "autoupdt".
```
user@system:~$cd ~
user@system:~$ls
```
Now that we know the file is there, we need to move the file to the proper directory. Type the following command at the command line promt to move the file:

user@system:~$sudo mv autoupdt /etc/cron.weekly
(You may be prompted to enter your sudo password, which is YOUR password.) Now we need to move to the Cron directory to verify the file was moved. At the command line prompt type:
```
user@system:~$cd /etc/cron.weekly
user@system:~$ls
```
You should see the "autoupdt" file in the list. (If not, try the previous move command again.)

Making the Cron Job File Executable
Now that the file is created and ready to be run weekly by cron, we still need to make the file executable in order for cron to be able to run it. Since you are already in the cron.weekly directory, all you need to do is enter this command at the prompt:
```
user@system:~$sudo chmod 755 autoupdt
```
(Again you may be prompted for your sudo password.)

Finished
Now that the file is executable, you are finished. The cron job will run weekly and update your source list (to see if there are any new package updates), update your packages as found and needed, then clean out any old unused no longer installed packages. You can still update using Synaptic Package Manager or apt-get at the command line, but now you can relax a bit knowing your system will be updated automatically on a weekly basis.

Advanced Alternative: Package Updates with Logging using Aptitude
Apt-get doesn't write details of changes it makes to log files. Aptitude logs to /var/log/aptitude, and can mail a report to root, which can be redirected to the sudo user if Postfix is set up to do this. As with the examples above, create this script as /etc/cron.weekly/autoupdt and make it executable. We use here a special action of aptitude called 'full-upgrade' (which will always upgrade your packages to the latest version, before Ubuntu 7.10 it was called 'dist-upgrade'). However, you might want to have a more conservative action when doing this on a server, the action 'safe-upgrade' is there for you. You can check the guide on aptitude for more information: AptitudeSurvivalGuide.
```
 #!/bin/bash

tmpfile=$(mktemp)

echo "aptitude update" >> ${tmpfile}
aptitude update >> ${tmpfile} 2>&1
echo "" >> ${tmpfile}
echo "aptitude full-upgrade" >> ${tmpfile}
aptitude -y full-upgrade >> ${tmpfile} 2>&1
echo "" >> ${tmpfile}
echo "aptitude clean" >> ${tmpfile}
aptitude clean >> ${tmpfile} 2>&1

mail -s "Aptitude cron $(date)" root < ${tmpfile}
rm -f ${tmpfile}
```
Here is a script (based on the above) that is useful if you don't have sendmail or postfix installed on your computer. It collects all the output from aptitude and manually sends an email using a mail server you specify. You must also specify the recipient of the email. As with the examples above, create this script as /etc/cron.weekly/autoupdt and make it executable:
```
 #!/bin/bash
#
# use aptitude to automatically install updates. log and email any
# changes.
#

#
# variables to change
#

# address to send results to
MAILTO=your@email.address
# host name of smtp server
MAIL=mail.yourisp.com


#
# script is below here (do not change)
#

tmpfile=$(mktemp)

#
# smtp setup commands
#

echo "helo $(hostname)" >> ${tmpfile}
echo "mail from: root@$(hostname)" >> ${tmpfile}
echo "rcpt to: $MAILTO" >> ${tmpfile}
echo 'data'>> ${tmpfile}
echo "subject: Aptitude cron $(date)" >> ${tmpfile}

#
# actually run aptitude to do the updates, logging its output
#

echo "aptitude update" >> ${tmpfile}
aptitude update >> ${tmpfile} 2>&1
echo "" >> ${tmpfile}
echo "aptitude full-upgrade" >> ${tmpfile}
aptitude -y full-upgrade >> ${tmpfile} 2>&1
echo "" >> ${tmpfile}
echo "aptitude clean" >> ${tmpfile}
aptitude clean >> ${tmpfile} 2>&1

#
# i get a lot of escaped new lines in my output. so the following
# removes them. this could be greatly improved

tmpfile2=$(mktemp)
cat ${tmpfile} | sed 's/\r\r/\n/g'|sed 's/\r//g' > ${tmpfile2}
mv ${tmpfile2} ${tmpfile}

#
# smtp close commands
#

echo >> ${tmpfile}
echo '.' >> ${tmpfile}
echo 'quit' >> ${tmpfile}
echo >> ${tmpfile}

#
# now send the email (and ignore output)
#

telnet $MAIL 25 < ${tmpfile} > /dev/null 2> /dev/null

#
# and remove temp files
#

rm -f ${tmpfile}
```
Creating weekly automatic upgrade script which tells if the upgrade was succesful

This script is designed to be scheduled to run with /etc/cron.* and it doesn't just send you the log file via email but tells you immediately if the upgrade was succesful or not; the temporary log file posted will be filtered for errors and warnings and if such things are found, the subject of the mail will be "Upgrade of your server failed".

It should be saved as /etc/cron.daily/autoupdate.sh or /etc/cron.weekly/autoupdate.sh depending on how often the upgrade needs to be done. When the script is saved in a proper location under /etc/cron.* it must be remembered to be given execution privileges (sudo chmod +x /etc/cron.*/autoupdate.sh). You also should set your email in the $admin_mail variable to make the mail feature working.

# This is a script made to keep your Ubuntu server up to
# date placing this script to /etc/cron.weekly/autoupdate.sh.
# This script updates your server automatically and informs
# you via email if the update was succesful or not.

# Set the variable $admin_email as your email address.
admin_mail="your.name@email.com"

# Create a temporary path in /tmp to write a temporary log
# file. No need to edit.
tmpfile=$(mktemp)

# Run the commands to update the system and write the log
# file at the same time.
```
echo "aptitupde update" >> ${tmpfile}
aptitude update >> ${tmpfile} 2>&1
echo "" >> ${tmpfile}
echo "aptitude full-upgrade" >> ${tmpfile}
aptitude -y full-upgrade >> ${tmpfile} 2>&1
echo "" >> ${tmpfile}
echo "aptitude clean" >> ${tmpfile}
aptitude clean >> ${tmpfile} 2>&1

# Send the temporary log via mail. The fact if the upgrade
# was succesful or not is written in the subject field.
if grep -q 'E: \|W: ' ${tmpfile} ; then
        mail -s "Upgrade of your server failed $(date)" ${admin_mail} < ${tmpfile}
else
        mail -s "Upgraded your server succesfully $(date)" ${admin_mail} < ${tmpfile}
fi

# Remove the temporary log file in temporary path.
rm -f ${tmpfile}
```
