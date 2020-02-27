---
layout: post
title: Configuring a shared media folder with Plex on FreeNAS
date: 2020-02-27 05:16:00 BRT
tags: freenas, plex, til
categories: articles
---

I struggled a little when configuring Plex + SMB on a ZFS dataset because there are no proper error messages for issues of this kind.

1. Create a new Dataset with share type `SMB` and sync `disabled`.
2. Go and edit ACL of this newly created dataset and click on the `Add ACL Item` button below
3. Set the _Who_ field to `everyone@` and all other fields should match the previous ones (you just need to set Permissions to `Full control`)
4. This step is not required for Plex. Create a SMB share and point it to the newly created Dataset.
5. Now go to the Plugins and install Plex. The installation is pretty straightforward and FreeNAS will download a bunch of files. Do not go to the UI yet.
6. Go to the Jails, select the one you've just created for Plex, and click Stop.
7. Now, with that selected, click on the right arrow to expand the item, then go to Mount Points.
8. Click on the menu Actions > Add, then select as source the Dataset you created on step 1, then as destination select `/mnt/Media`
9. Now you can start the Jail and proceed with the Plex configuration.
10. When asked to add your media library, just locate them under the `/mnt/Media` folder.

As always, I didn't figure this out on my own. Thanks to [jnkbcs](https://forums.plex.tv/t/adding-folders-and-last-folder-is-greyed-out/478088/12) for digging into this issue, you saved me lots of time!
