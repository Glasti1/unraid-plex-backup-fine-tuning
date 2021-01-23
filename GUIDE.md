# UnRAID Plex Backup - Fine Tuning

Guide to separate the "important" and "less important" folders from the Plex appdata folder to make backups faster and reduce Plex downtime to a minimum.

**Disclaimer: I am not responsible for any possible data loss by following this guide. Use at your own risk.
This backup method has been tested by several users and confirmed working.**

## Assumption

[Hotio's image](https://github.com/hotio/plex) of the Plex container is being used - if this is not the case you will need to confirm the correct paths for your container.

## Background

The Plex appdata folder can get rather big if you have a lot of content and all features enabled such as video thumbnails.
Also the Plex app data folder contains data that is important, for Plex to run properly, which you want to backup daily and less important data, for user experience, that you can backup less frequently.
**This guide is NOT a complete replacement for your regular backups, it will NOT restore your Plex container if it breaks. Keep backing up all your container appdata, including Plex, normally following the 1-2-3 strategy.**

A great way to backup your appdata folder is using [IamSpartacus' script](https://github.com/SpartacusIam/unraid-scripts/blob/master/backup_all_appdata).

**A big thank you to IamSpartacus for writing these scripts to make this possible. If you like his work, please [buy him a beer!](https://www.buymeacoffee.com/iamspartacus)**

## Configuration

First lets have a look at the scripts. You can find them [here](https://github.com/SpartacusIam/unraid-scripts)

We will be using the following scripts:

* [Backup Plex databases](https://github.com/SpartacusIam/unraid-scripts/blob/master/backup_plex_dbs)

  * This will backup the important Plex database data

* [Backup Plex appdata](https://github.com/SpartacusIam/unraid-scripts/blob/master/backup_plexdata)

  * This will backup the 'less' important Plex media and metadata data

We will be using the plugin called `User Scripts` which you can install through the `Community Applications` plugin.

**Backup your Plex app data folder in case something goes wrong.**

1. Let's find the paths to the following folders within you Plex appdata folder:

    * These are the "less" important folders we are going to move outside of the appdata folder.
        * `/Cache/Transcode/Sync+`
        * `/Cache/PhotoTranscoder`
        * `/Media`
        * `/Metadata`

    * These are the critical files that we are not moving, but we will need the paths later for the backup script.
        * `/Databases/com.plexapp.plugins.library.db`
        * `/Databases/com.plexapp.plugins.library.blobs.db`

    In my case these folders are under `/mnt/disks/plex/appdata/plex/app/Plex Media Server/*`.

    > Note: `/mnt/disks/*` refers to an unassigned mounted drive. Unassigned drives are not protected by the parity, like cache pools.
    >
    > Not every repository will have the same paths. The paths shown above may not work for you, and you will have to confirm the correct paths.

2. Create the following new folders outside of the appdata folder:

    * `/Cache/Transcode/Sync+`
    * `/Cache/PhotoTranscoder`
    * `/Media`
    * `/Metadata`

    In my case I place them all under `/mnt/disks/plex/plex_data/*`

    > Note: Match these paths to your setup.
    >
    > Always make sure you use the real paths. For example. If the `plex_data` folder lives on a cache pool, using `/mnt/user/plex_data/*` will most likely not work. You will need `/mnt/<name_of_your_cache_pool>/plex_data/*`.

3. Stop the Plex container.

4. Move all the content inside the folders from Step 1 to the newly created folders in Step 2.
    **Do not delete the folders or contents from the appdata folder.**

5. While the files are moving we are going to create new paths within the container config to match the newly created folders.

    * Go to the Plex container config
    * Select `Add another Path, Port, Variable, Label or Device`
    * Config type: `Path`
    * Name: `sync+`
    * Container Path: `/config/app/Plex Media Server/Cache/Transcode/Sync+`

    > Note: Match these paths to your setup.

    * Host Path: `/mnt/disks/plex/plex_data/Cache/Transcode/Sync+`

    > Note: Match these paths to your setup

    * Press `ADD`
        * **Repeat this step for the PhotoTranscoder, Media and Metadata paths**
    * Press `APPLY` to save the new config.

    >Note: when using an unassigned drive for your data do not forget to use `R/W Slave`

6. Now let's setup the scripts.

    * To backup the 'less' important Plex data:
        * Go to the user scripts plugin, *Settings --> User Plugins*
        * Click `ADD NEW SCRIPT` and give it a name. I call it `backup_plexdata`.
        * Edit the script and paste [this script](https://github.com/SpartacusIam/unraid-scripts/blob/master/backup_plexdata) in there.

    > NOTE: You only need 1 `#!/bin/bash`, otherwise it won't work.

    * Change the following paths to the match your paths:
        * `plexdataDirectory='/PATH/TO/YOUR/PLEXDATA/'`
        * `backupDirectory='/PATH/TO/YOUR/BACKUP/DIRECTORY/'`

    In my case it looks like this:

    ```bash
        plexdataDirectory='mnt/disks/plex/plex_data/'
        backupDirectory='/mnt/user/backups/plex/'
    ```

    * Press `SAVE CHANGES`

    * To backup the important Plex data:
        * Click `ADD NEW SCRIPT` and give it a name. I call it `backup_plex_dbs`
        * Edit the script and paste [this script](https://github.com/SpartacusIam/unraid-scripts/blob/master/backup_plex_dbs) in there.

    > NOTE: you only need 1 `#!/bin/bash`, otherwise it won't work.

    * Change the following paths to the match your paths:
        * `plexdbDirectory='/PATH/TO/YOUR/PLEX/DATABASES/DIRECTORY/'`
        * `backupDirectory='/PATH/TO/YOUR/BACKUP/DIRECTORY/'`

    In my case it looks like this:

    ```bash
        plexdbDirectory='/mnt/disks/plex/appdata/plex/app/Plex Media Server/Plug-in Support/Databases/'
        backupDirectory='/mnt/user/backups/plex/plexdb/'
    ```

    * Press `SAVE CHANGES`

7. Now we need to set a schedule, I prefer to do this through a crontab. I use [this website](https://crontab.guru/) to create a crontab.

    For `backup_plexdata` I have it set as follows:

    ```bash
    30 2 * * 2,6
    ```

    * This backup runs at 02:30AM on Tuesday and Friday

    For `backup_plex_dbs` I have it set as follows:

    ```bash
    10 2 * * *
    ```

    * This backup runs runs at 02:10AM every night.

8. Test if the new paths are working. If you start Plex and all your metadata is showing it is working as intended!

9. Test if the backup scripts are working properly. Enjoy your fast backups with minimal downtime!

If you have any questions, you can find me in the UnraidOfficial and [UnraidCommunity](https://discord.gg/qWPbc8R) servers on Discord!
