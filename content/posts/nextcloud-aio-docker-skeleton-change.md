---
title: "Changing the default files and folders in the Nextcloud All-in-One Docker Image"
date: 2023-11-23T08:16:16Z
tags: ["Nextcloud", "Docker"]
categories: ["SysAdmin"]
description: "Worked example of how to change the default skeleton files and folders viewable to a new user in the Nextcloud All-in-One Docker Image"
enableToc: false
draft: false
---

## Erratum

[Nextcloud](https://nextcloud.com/) is an open-source collaboration platform where teams can share files, calendars and other resources. When a system administrator creates a new user on Nextcloud, the default files and folders that user views on their initial login are automatically populated as part of the Nextcloud setup (e.g. a pdf file entitled *Reasons to use Nextcloud*, etc).  As the administrator you may want your users to view a different set of files and folders; and if you have installed Nextcloud via the [All-in-One Docker Image](https://github.com/nextcloud/all-in-one), this has to be done whilst navigating a docker container.

This was originally a worked example demonstrating how to change these default files and folders (also called the skeleton directory), based on the already existing Nextcloud tutorial on [providing default files](https://docs.nextcloud.com/server/latest/admin_manual/configuration_files/default_files_configuration.html).

However shortly after publication it [was pointed out](https://help.nextcloud.com/t/worked-example-of-changing-the-skeleton-default-files-and-folders-in-the-nextcloud-all-in-one-docker-image/175146/2?u=gene_preciouschicken) that my solution was both:

-  wrong and
-  already covered in the documentation.

Therefore if you are looking to do this please see the [custom skeleton directory](https://github.com/nextcloud/all-in-one#custom-skeleton-directory) section of the Nextcloud AIO Docker FAQ.  Sorry for the inconvenience.
