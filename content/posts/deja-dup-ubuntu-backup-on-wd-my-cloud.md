---
title: "Configuring Deja Dup Ubuntu Backup on a WD My Cloud NAS"
date: 2019-12-04T11:05:57Z
tags: ["Linux", "Deja Dup Backup"]
draft: false
---

Considering that the sole purpose of [Deja Dup](https://wiki.gnome.org/Apps/DejaDup) is to backup and one of the main selling points of a WD My Cloud NAS drive is as a place to store your backup; it is surprisingly difficult to do it in a seamless way on Ubuntu.  By seamless I mean it happens in the background without you having to think about it.

Here is how I did it on Ubuntu 18.04 LTS with a WD My Cloud Mirror:

---

### 1. Create a share on WD My Cloud  

Using the My Cloud interface create a 'share' on the WD My Cloud (see About Shares in the [User Manual](https://products.wdc.com/library/UM/ENG/4779-705145.pdf)), ensure NFS is set to on and give the share a relevant name (e.g. Backup).

Once created take a note of the IP address of the share as shown in the screenshot (here it is 192.168.0.32, your system will likely be different):

[![WD MyCloud Share Access](https://www.preciouschicken.com/blog/images/share_access.png)](https://www.preciouschicken.com/blog/images/share_access.png.png)

### 2. Create a local folder to act as mount point

Using the terminal create a folder on the computer you are backing up: `sudo mkdir /mnt/Backup`.  This is the folder you will mount the NAS drive too; you can create this folder anywhere you consider sensible.

### 3. Open *fstab* file 

Using the terminal open your fstab file using `sudo vim /etc/fstab`. This fstab file determines what drive Ubuntu mounts at startup.

### 4.  Add new line to *fstab* file

Create a new line at the end of the *fstab* file and; where '192.168.0.32' is your IP address from Step 1, the first `Backup` is the share you created at Step 1 and `/mnt/Backup` is the folder you created at Step 2; add the following 

```bash
192.168.0.32:/nfs/Backup /mnt/Backup nfs defaults 0 0
```

If you aren't familiar with vim it can be a little tricky to figure out how to edit text and save, but you can [pick up the bare minimum quickly](https://yos.io/2013/07/10/learn-vim-in-5-minutes/).

### 5.  Create */etc/network/if-up.d/fstab* file  

Using the terminal create a file with the command `sudo vim /etc/network/if-up.d/fstab`.

### 6.  Edit */etc/network/if-up.d/fstab* file

Copy and paste the following text to the file created at Step 5:

```bash
#!/bin/sh
mount -a
```

### 7.  Make */etc/network/if-up.d/fstab* file executable

Make the file executable: `sudo chmod +x /etc/network/if-up.d/fstab`.

### 8.  Reboot.  

Reboot.  The drive should mount on startup.

### 9.  Select backup options in Deja Dup

Open Backups on Ubuntu (although the name is Deja Dup, it is not known as that) and select 'Storage location'.  For storage location select 'Local Folder' and then choose */mnt/Backup* or whatever folder you chose at Step 2.

### 10.  Back up  

Select 'Back Up Now' from Overview and set up scheduling.

---

These instruction borrow heavily from an [answer on Ubuntu Forums](https://ubuntuforums.org/showthread.php?t=2392742&p=13795542#post13795542), which was helpfully [reposted on Ask Ubuntu](https://askubuntu.com/a/1153732).  Linuxize also has a [useful post on mounting NFS file systems](https://linuxize.com/post/how-to-mount-an-nfs-share-in-linux/).

**Addendum**: The original version of this post used the device name (see Configuring Settings in the [User Manual](https://products.wdc.com/library/UM/ENG/4779-705145.pdf)) instead of the IP address in the fstab file.  For some reason that I'm not going to dig too much into, this has recently stopped working; so I'm using the IP address instead.  Using the IP address has problems too - as they are normally dynamically allocated by a router they tend to change when devices are powered down / reset; in which case the fstab above will need editing.  I figure as most NAS drive are on permanently however, this is not too big a hassle - or alternatively you can amend your router to provide a static IP addresses which doesn't change.

