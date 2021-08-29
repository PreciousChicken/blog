---
title: "Automating restic backups from a Raspberry Pi to a WD My Cloud NAS"
date: 2021-08-29T08:05:25+01:00
tags: ["Linux", "Restic", "NAS", "Backup"]
categories: ["SysAdmin"]
description: "How to set up automated restic backups from a Raspberry Pi to a WD My Cloud NAS"
enableToc: true
draft: true
---

I've blogged previously on using [Deja Dup to backup Ubuntu Linux to a NAS drive](https://www.preciouschicken.com/blog/posts/deja-dup-ubuntu-backup-on-wd-my-cloud/).  However I found using the same strategy on a Raspberry Pi did not work so well; the Pi kept on forgetting the backup password meaning it was hard to automate and encrypt backups.  Although a solvable problem, I thought I would take a look at restic instead as a solution.  The article below features a how-to on how this was automated written for the [WD My Cloud NAS](https://amzn.to/2MsLti5) drive[^1] using NFS (Network File System) as the file system protocol.

[^1]: Amazon affiliate link.

## Create a share on WD My Cloud  

Using the My Cloud interface create a 'share' on the WD My Cloud (see About Shares in the [User Manual](https://products.wdc.com/library/UM/ENG/4779-705145.pdf)), ensure NFS is set to on and give the share a relevant name (e.g. Backup).  Once created take a note of the IP address of the share as shown in the screenshot below (here it is 192.168.0.32, your system will likely be different).  If you are running the current My Cloud OS 5 then you will see:

![WD MyCloud Share Access OS 5](https://www.preciouschicken.com/blog/images/share_access_5.png)

In this version of the OS you will need to explicitly enable Write via the *Configure >>* link:

![WD MyCloud Configure NFS](https://www.preciouschicken.com/blog/images/nfs_configure.png)

The Host value refers to the IP address on your local network that can connect to the share - leave as asterisk to enable all.  Alternatively add the IP address of the machine you are backing up from to restrict access.

## Install the NFS client on the Raspberry Pi

The remained of the instructions are now based within the Raspberry Pi using the command line.  Install the NFS client package by running the following at the terminal:

```bash
sudo apt update
sudo apt install nfs-common
```

## Create a local folder to act as mount point

Using the terminal create a folder on the computer you are backing up: `sudo mkdir /mnt/Backup`.  This is the folder you will mount the NAS drive too; you can create this folder anywhere you consider sensible.

## Open *fstab* file 

Using the terminal open your fstab file using `sudo vim /etc/fstab`. This fstab file determines what drive Ubuntu mounts at startup.

## Add new line to *fstab* file

Create a new line at the end of the *fstab* file and; where '192.168.0.32' is your IP address from Step 1, the first `Backup` is the share you created at Step 1 and `/mnt/Backup` is the folder you created at Step 2; add the following 

```bash
192.168.0.32:/nfs/Backup /mnt/Backup nfs defaults 0 0
```

If you aren't familiar with vim it can be a little tricky to figure out how to edit text and save, but you can [pick up the bare minimum quickly](https://yos.io/2013/07/10/learn-vim-in-5-minutes/).

## Create */etc/network/if-up.d/fstab* file  

Using the terminal create a file with the command `sudo vim /etc/network/if-up.d/fstab`.

## Edit */etc/network/if-up.d/fstab* file

Copy and paste the following text to the file created at Step 5:

```bash
#!/bin/sh
mount -a
```

## Make */etc/network/if-up.d/fstab* file executable

Make the file executable: `sudo chmod +x /etc/network/if-up.d/fstab`.

## Mount the drive  

Run `sudo mount -a` from the terminal and the drive should now mount.  If you have done something wrong, like entered the wrong IP in *fstab*, you will get a message similar to `mount.nfs: Network is unreachable`; success meanwhile is marked simply by the lack of error message.  Alternatively you could also reboot your system which has the same effect - although you don't get to see the error message immediately if there is one.

## Initiate restic repo

Having specified where we are going to host the backup, it is time to initiate the repository restic will use to store our backup on the NAS drive.

At the command line enter:

```bash
restic init --repo /mnt/Backup
```

If successful restic will prompt you for a password to encrypt your repository and then create a new repository in your mounted backup directory.

## Provide restic with a password

When the repo was created a password was provided at the command line - however if this process is automated this password needs to be stored somewhere so it can be accessed by the automated script.

Changing *your_password* as required enter the following at the terminal:

```bash
echo "your_password" > .config/restic-backup-password.txt
chmod 600 .config/restic-backup-password.txt
```

This creates a text file containing your password, then alters the permissions for that file so only the current user can read it.

## Add restic commands to Crontab

Having 
