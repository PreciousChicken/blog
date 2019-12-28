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

### 2. Create a local folder to act as mount point

Using the terminal create a folder on the computer you are backing up: `sudo mkdir /mnt/Backup`.  This is the folder you will mount the NAS drive too; you can create this folder anywhere you consider sensible.

### 3. Open *fstab* file 

Using the terminal open your fstab file using `gksudo gedit /etc/fstab`. This fstab file determines what drive Ubuntu mounts at startup.

### 4.  Add new line to *fstab* file

Create a new line at the end of the *fstab* file and; where 'wdmycloudmirror' is your device name (see Configuring Settings in the [User Manual](https://products.wdc.com/library/UM/ENG/4779-705145.pdf)), the first `Backup` is the share you created at Step 1 and `/mnt/Backup` is the folder you created at Step 2; add the following 

```bash
wdmycloudmirror.local:/nfs/Backup /mnt/Backup nfs defaults 0 0
```

### 5.  Create */etc/network/if-up.d/fstab* file  

Using the terminal create a file with the command `gksudo gedit /etc/network/if-up.d/fstab`.

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

These instruction borrow heavily from an [answer on Ubuntu Forums](https://ubuntuforums.org/showthread.php?t=2392742&p=13795542#post13795542), which was helpfully [reposted on Ask Ubuntu](https://askubuntu.com/a/1153732).

