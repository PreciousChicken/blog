---
title: "Automating restic backups from a Raspberry Pi to a WD My Cloud NAS"
date: 2021-08-29T08:05:25+01:00
tags: ["Linux", "Restic", "NAS", "Backup", "Raspberry Pi"]
categories: ["SysAdmin"]
description: "How to set up automated restic backups from a Raspberry Pi to a WD My Cloud NAS"
enableToc: true
draft: true
---

I've blogged previously on using [Deja Dup to backup Ubuntu Linux to a NAS drive](https://www.preciouschicken.com/blog/posts/deja-dup-ubuntu-backup-on-wd-my-cloud/).  However I found using the same strategy on a Raspberry Pi did not work so well; the Pi kept on forgetting the backup password meaning it was hard to automate and encrypt backups.  Although a solvable problem, I thought I would take a look at restic instead as a solution to back up a [Tiddlywiki hosted on a Raspberry Pi](https://preciouschicken.com/blog/posts/tiddlywiki5-raspberry-pi-guide/).  The article below features a how-to on how this was automated written for the [WD My Cloud NAS](https://amzn.to/2MsLti5) drive[^1] using NFS (Network File System) as the file system protocol.

[^1]: Amazon affiliate link.

## Create a share on WD My Cloud  

Using the My Cloud interface create a 'share' on the WD My Cloud (see About Shares in the [User Manual](https://products.wdc.com/library/UM/ENG/4779-705145.pdf)), ensure NFS is set to on and give the share a relevant name (e.g. Backup).  Once created take a note of the IP address of the share as shown in the screenshot below (here it is 192.168.0.32, your system will likely be different).  If you are running the current My Cloud OS 5 then you will see:

![WD MyCloud Share Access OS 5](https://www.preciouschicken.com/blog/images/share_access_5.png)

In this version of the OS you will need to explicitly enable Write via the *Configure >>* link:

![WD MyCloud Configure NFS](https://www.preciouschicken.com/blog/images/nfs_configure.png)

The Host value refers to the IP address on your local network that can connect to the share - leave as asterisk to enable all.  Alternatively add the IP address of the machine you are backing up from to restrict access.

## Install the NFS client on the Raspberry Pi

Turning to the Raspberry Pi for the remainder of these instructions, first the NFS client package should be installed using the following command at the terminal:

```bash
sudo apt update
sudo apt install nfs-common
```

## Create a local folder to act as mount point

Next create a local folder we will back up to: `sudo mkdir /mnt/Backup`.  This is the folder you will mount the NAS drive to; you can create this folder anywhere you consider sensible.

## Edit *fstab* file 

Using the your favourite text editor open the *fstab* file with root privileges e.g. at the terminal type `sudo vim /etc/fstab`. This *fstab* file determines what drive the Pi mounts at startup.

Create a new line at the end of the *fstab* file and; where '192.168.0.32' is the IP address of the NAS, the first `Backup` is the share created on your NAS and `/mnt/Backup` is the local folder created; add the following 

```bash
192.168.0.32:/nfs/Backup /mnt/Backup nfs defaults 0 0
```

If aren't familiar with vim and you are using it to edit from the command line, it can be a little tricky to figure out how to edit text and save, but you can [pick up the bare minimum quickly](https://yos.io/2013/07/10/learn-vim-in-5-minutes/).

## Mount the drive  

Run `sudo mount -a` from the terminal and the drive should now mount.  If you have done something wrong, like entered the wrong IP in *fstab*, you will get a message similar to `mount.nfs: Network is unreachable`; success meanwhile is marked simply by the lack of error message.  

## Edit Pi Config

In theory editing the *fstab* should allow the drive to be mounted every time the Pi is booted, but a further modification is required to do this (as discovered via a StackOverflow [answer](https://raspberrypi.stackexchange.com/a/53147/138949)).  At the command line enter the Pi configuration by running:

```bash
sudo raspi-config
```

Then select the following options: *1 System Options* - *S6 Network at Boot* and in answer to the question 

> Would you like boot to wait until a network connection is established?

select *Yes*.  If all went well the Pi confirms:

> Waiting for network on boot is enabled

OR EDIT:

Have to run this command on Manjaro, works on PI????
sudo systemctl enable systemd-networkd-wait-online
https://forum.manjaro.org/t/fstab-mounting-network-drive-on-boot-not-working/53699/4

## Initiate restic repo

Having specified where we are going to host the backup, it is time to initiate the repository restic will use to store our backup on the NAS drive.

At the command line enter:

```bash
restic init --repo /mnt/Backup/wiki-restic-repo
```

If successful restic will prompt you for a password to encrypt your repository and then create a new repository in your mounted backup directory.

## Provide restic with a password

When the repo was created a password was provided at the command line - however if this process is automated this password needs to be stored somewhere so it can be accessed by the automated script.

Changing *your_password* as required enter the following at the terminal:

```bash
echo "your_password" > ~/.config/restic-backup-password.txt
chmod 600 ~/.config/restic-backup-password.txt
```

This creates a text file containing your password, then alters the permissions for that file so only the current user can read it.

## Add restic commands to Crontab

Having initated a restic repository we now need to automate the backup process using cron, which allows a user to schedule tasks to run on a periodic basis.

In the terminal enter `crontab -e` which will prompt you to choose a text editor to alter your crontab.  At the bottom of this file enter two new lines:

```bash
@hourly /usr/bin/restic -r /mnt/Backup/wiki-restic-repo/ backup /home/pi/wiki/ --tag auto --password-file=/home/pi/.config/restic-backup-password.txt
@daily /usr/bin/restic forget -r /mnt/Backup/wiki-restic-repo/ --password-file=/home/pi/.config/restic-backup-password.txt --keep-last 8 --keep-daily 5 --keep-weekly 4 --keep-monthly 10 --keep-yearly 5 --tag auto --prune
```

The first of these lines backs-up the chosen directory every hour, the second line prunes the repository so that it doesn't grow to an unmanageable size.

## Further resources

-  [How To Use Cron to Automate Tasks on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-use-cron-to-automate-tasks-ubuntu-1804).  Excellent tutorial from Digital Ocean (as many of them are).
-  [Restic documentation](https://restic.readthedocs.io/en/stable/).  Official documentation that is comprehensive and easy to follow.
