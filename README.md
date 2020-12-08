# unraid-plex-backup-fine-tuning
Guide to seperate the 'important' and 'less important' folders from the plex appdata to make backups fasters and reduce plex downtime to a minimum


**Disclaimer: I am not responsible for any possible data loss by following this guide. Use at your own risk. 
This backup method has been tested by several users and confirmed working.**


> The Plex app data folder can get rather big if you have a lot of content and all features, like video thumbnails, enabled. 
Also the Plex app data folder contains data that is important, for Plex to run properly, which you want to backup daily and less important data, for user experience, that you can backup less frequently.
> **This guide is NOT a complete replacement for your regular backups, it will NOT restore your plex container if it breaks. Keep backing up all your container app data, including plex, normally following the 1-2-3 strategy.** 
> A great way to backup all app data is using IamSpartacus [backup_all_appdata](https://github.com/SpartacusIam/unraid-scripts/blob/master/backup_all_appdata) script.

In this guide i will show you how to make your backups smaller, faster and reduce Plex downtime to an minimum.

**A big thank you to IamSpartacus for writing these scripts to make this possible. If you like his work, please [buy him a beer!](https://www.buymeacoffee.com/iamspartacus)**

Let's get started

First lets have a look at the scripts. You can find them at https://github.com/SpartacusIam/unraid-scripts

We will be using the following scripts:

- https://github.com/SpartacusIam/unraid-scripts/blob/master/backup_plex_dbs 

> This will backup the important Plex database data

- https://github.com/SpartacusIam/unraid-scripts/blob/master/backup_plexdata 

> This will backup the 'less' important Plex media and metadata data

Also we will be using the plugin called `User Scripts` which you can install through the `Community Applications` plugin. 

**1.** 

Backup your Plex app data folder for if something goes wrong.

**2.**

You want to find the paths to the following folders within you Plex app data folder:
- `/Cache/Transcode/Sync+`
- `/Cache/PhotoTranscoder`
- `/Media`
- `/Metadata`

> These are the 'less' important folders we are going to move outside of the
> app data folder.

- `/Databases/com.plexapp.plugins.library.db`
- `/Databases/com.plexapp.plugins.library.blobs.db`

> These are critical files we are not moving, but we will need the paths
> later for the backup script

In my example these folders are under `/mnt/disks/plex/appdata/plex/app/Plex Media Server/*`

**3.**

Create new the following folders outside of the app data folder.

- `/Cache/Transcode/Sync+` 
- `/Cache/PhotoTranscoder` 
- `/plex_data/Media` 
- `/Metadata` 

In my example i place them all under `/mnt/disks/plex/plex_data/*`

> Note: `match these paths to your setup`

**4.**

Stop the Plex container.

**5.**

Move all the content inside the folders from step **3** to the new created folders in step **4**.

**6.**

While the files are moving we are going to create new paths within the container config to match the new created folders.

- Go to the Plex container config
- Select `Add another Path, Port, Variable, Label or Device`
- Config type: `Path`
- Name: `sync+`
- Container Path: `/config/app/Plex Media Server/Cache/Transcode/Sync+/ `

> Note: match these paths to your setup

- Host Path: `/mnt/disks/plex/plex_data/Cache/Transcode/Sync+/ `

> Note: match these paths to your setup

- Press `ADD`
- **Repeat this step for the PhotoTranscoder, Media and Metadata paths**
- Press `APPLY` to save the new config.

**7.**

Now let's setup the scripts.

To backup the 'less' important Plex data:

- Go to the user scripts plugin, *Settings --> User Plugins*
- Click `ADD NEW SCRIPT` and give it a name. I call it `backup_plexdata`.
- Copy and paste the this script in there. 
https://github.com/SpartacusIam/unraid-scripts/blob/master/backup_plexdata

> NOTE: you only need 1x `#!/bin/bash`, otherwise it wont work

- Change the following paths to the match your paths:

 `#Set Plexdata Directory (must include trailing /)

plexdataDirectory='/PATH/TO/YOUR/PLEXDATA/'`

In my example `/mnt/disks/plex/plex_data/ `


and 

 
`#Set Backup Directory (must include trailing /)

backupDirectory='/PATH/TO/YOUR/BACKUP/DIRECTORY/' `

My backups go to `/mnt/user/backups/plex/`


- Press `SAVE CHANGES`

To backup the important Plex data:

- Click `ADD NEW SCRIPT` and give it a name. I call it `backup_plex_dbs`
- Copy and paste the this script in there. 
https://github.com/SpartacusIam/unraid-scripts/blob/master/backup_plex_dbs

> NOTE: you only need 1x `#!/bin/bash`, otherwise it wont work

- Change the following paths to the match your paths:
 
`#Set Appdata Directory (must include trailing /)

plexdbDirectory='/PATH/TO/YOUR/PLEX/DATABASES/DIRECTORY/'`

In my example `/mnt/disks/plex/appdata/plex/app/Plex\ Media\ Server/Plug-in\ Support/Databases/`

`#Set Backup Directory (must include trailing /)

backupDirectory='/PATH/TO/YOUR/BACKUP/DIRECTORY/'`

My backups go to `/mnt/user/backups/plex/plexdb/`

- Press `SAVE CHANGES`

**8.**

Now we need to set a schedule, i prefer to do this through a crontab. I use this website to create a crontab. https://crontab.guru/

For `backup_plexdata` i have set `30 2 * * 2,6`. 

> The backup runs at 02:30AM on Tuesday and Friday

For `backup_plex_dbs` i have set `10 2 * * *`. 

> The backup runs runs at 02:10AM every night.

**9.**

Test the scripts if they are working properly.

Enjoy your fast backups with minimal downtime. 

