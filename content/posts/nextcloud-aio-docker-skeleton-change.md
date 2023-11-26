---
title: "Changing the default files and folders in the Nextcloud All-in-One Docker Image"
date: 2023-11-23T08:16:16Z
tags: ["Nextcloud", "Docker"]
categories: ["SysAdmin"]
description: "Worked example of how to change the default skeleton files and folders viewable to a new user in the Nextcloud All-in-One Docker Image"
enableToc: true
draft: false
---

## Introduction

[Nextcloud](https://nextcloud.com/) is an open-source collaboration platform where teams can share files, calendars and other resources. When a new user on Nextcloud is created by a system administrator, when they first log onto the system they are presented with default files and folders automatically pre-generated (e.g. a pdf file entitled *Reasons to use Nextcloud*, etc).  As the administrator you may want your users to view a different set of files and folders; and if you have installed Nextcloud via the [All-in-One Docker Image](https://github.com/nextcloud/all-in-one), this has to be done whilst navigating a docker container.

This is a worked example demonstrating how to change these default files and folders (also called the skeleton directory), based on the already existing Nextcloud tutorial on [providing default files](https://docs.nextcloud.com/server/latest/admin_manual/configuration_files/default_files_configuration.html).  The versions used in this guide are Nextcloud AIO v7.6.2 installed on Ubuntu Linux 22.04.3 LTS.  If you have installed on a non-Linux server, this guide be less useful...

## Opening a secure shell

We need initially to connect to our remote server hosting our Nextcloud installation; clearly this will vary considerably depending on how you normally connect and where you have hosted it, but typically this is done via a secure shell (`ssh`) and the terminal command on Linux to do so will look a little like this:

```bash
ssh ubuntu@192.168.1.33
```

Clearly change for your circumstances!

DigitalOcean provide a great [overview of connecting with ssh](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys#basic-connection-instructions).

## Enter the docker container

The Nextcloud AIO installation will have started a number of docker containers on your server, we will now enter the one containing the *skeleton* directory where the default files are kept:

```bash
sudo docker exec -it nextcloud-aio-nextcloud bash
```

After probably being prompted for your password you will enter the container and your terminal prompt will likely change to something resembling `8eg458130ed6:/var/www/html#`.

## Create a new skeleton folder

The default files and folders that Nextcloud uses can be viewed in the `core/skeleton` directory, we need to create now going to create a new skeleton directory to hold our new resources:

```bash
mkdir core/admin_skeleton
```

NB - Making amends to the original skeleton directory will not work as this will get overwritten by the system.

## Populate the skeleton folder

At this point you decide what you wish to populate the skeleton folder with; for instance new folders (e.g. `mkdir core/admin_skeleton/MyNewFolder`) or files (e.g. `vi core/admin_skeleton/Readme.md` to draft a new Readme file).  Should you want to import folders that you have created elsewhere you will first have to import them from your local machine onto the server, potentially using SCP (Secure Copy Protocol) as explained in DigitalOcean's [Ultimate Guide to SCP](https://snapshooter.com/learn/linux/copy-files-scp), and then transfer them from the server into the docker container using the [docker cp](https://docs.docker.com/engine/reference/commandline/cp/) command.

It is also possible to leave the folder empty.

## Change ownership

Once we have finished making our changes; we need to adjust the file ownership and permissions of anything we have created, in accordance with the [Nextcloud manual](https://docs.nextcloud.com/server/latest/admin_manual/maintenance/manual_upgrade.html).

Firstly we need to  change the ownership of the created folder (amending the command to your own value if you did not use *admin_skeleton*):

```bash
sudo chown -R www-data:www-data /var/www/html/core/admin_skeleton
```

And then we need to change permissions, for the skeleton folder (and any other folder you might have created), as follows:

```bash
sudo chmod 755 /var/www/html/core/admin_skeleton
```

Should you have created any files then they need to have their permissions adjusted as shown:

```bash
sudo chmod 644 /var/www/html/core/admin_skeleton/Readme.md
```

We are now finished within the docker container itself so we can exit the container with:

```bash
exit
```

NB - The 755 / 644 permissions choice suggested above are based on the current value of the files and folders within the skeleton folder, plus a [howto from LightningCR](https://lightningcr.com/projects/nextcloud/wiki/Creating_a_Custom_Skeleton_Directory_for_Nextcloud).  This does seem to contradict however [the manual](https://docs.nextcloud.com/server/latest/admin_manual/maintenance/manual_upgrade.html) and a [question on ownership of files on the Nextcloud forum](https://help.nextcloud.com/t/change-ownership-of-nextcloud-files-root-to-www-data/150657).  If I find a definitive answer I'll update this guide...

## Scan the new files

As we've been manually created new files it probably is not a bad idea[^1] to run the [occ file:scan](https://docs.nextcloud.com/server/15/admin_manual/configuration_server/occ_command.html?highlight=occ#scan) command to make sure everything has been found and indexed:

```bash
sudo docker exec --user www-data -it nextcloud-aio-nextcloud php occ files:scan --all
```

[^1]: In truth I think this process could be run without this command as technically the files and directories we have created aren't 'user' files, but this step is probably a good idea in case we have missed something...

## Update config file

Next we have to update the configuration file to point to our new skeleton folder.  Again we can use *occ* to do this with the [config:system:set](https://docs.nextcloud.com/server/15/admin_manual/configuration_server/occ_command.html?highlight=occ#setting-a-single-configuration-value) command.  Ensuring you have the right absolute path to your new skeleton directory as the value enter:

```bash
sudo docker exec --user www-data nextcloud-aio-nextcloud php occ config:system:set skeletondirectory --value="/var/www/html/core/admin_skeleton"
```

This being the last step we can now leave the server by again:

```bash
exit
```

## Conclusion

And we are done.  Any new users created should see the new skeleton folder you have created for them by default (users created before this change may be left viewing the old folders too...).

It is worth adding that any folders you create will be individual folders for that user only; so if you create a folder named *UsefulDocs* and User A puts a file in that, User B will not be able to view it by default.  If you are looking to create folders that more than one user can use then take a look at the [Group folders](https://apps.nextcloud.com/apps/groupfolders) application.

Also worth mention is that currently the *Templates* folder is always created for new users, whether you want it or not; as of writing there is a current [feature / bug issue](https://github.com/nextcloud/server/issues/39266) to disable this on github.

Like the post?  Comments / feedback?  Let me know below.

