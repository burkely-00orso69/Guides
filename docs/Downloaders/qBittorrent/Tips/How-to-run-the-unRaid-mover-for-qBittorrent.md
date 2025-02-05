# How to run the unRaid mover for qBittorent seeding torrents

--8<-- "includes/downloaders/basic-setup.md"

When you make use of the unRaid cache drive for your `/data/torrents` share and the torrents in qBittorent are still seeding then the mover can't move files, because they are still in use.

Using the following instructions you will be able to move the files with the use of the qBittorrent API.

??? summary "Workflow Rules - [CLICK TO EXPAND]"

    1. Pause torrents older than last x days.
    1. Run the mover.
    1. Resume the torrents once the mover is completed.

!!! Danger
    If you're going to make use of this make sure you're using `Post-Import Category` in your Starr apps [^1] Download clients settings, especially when you're using the Seed Time/Ratio settings in your Indexers settings in the Starr apps.

    Else it could happen when the torrents get paused that they get removed by the Starr apps before the seeding goal is reached :bangbang:

## Needed

### The Script

Download the following standalone script.

- [Script](https://raw.githubusercontent.com/StuffAnThings/qbit_manage/master/scripts/mover.py){:target="_blank" rel="noopener noreferrer"}

Big Thnx to [bobokun](https://github.com/bobokun){:target="_blank" rel="noopener noreferrer"} Developer of [qBit Manage](https://github.com/StuffAnThings/qbit_manage){:target="_blank" rel="noopener noreferrer"}

### Plugins

Install the following Plugins.

- User Scripts
- Nerd Tools
  - python3
  - python-setuptools

------

## Setup

After you installed the needed Plugins it's time to configure everything.

### qBit API

The script needs the qBit API to work, so we need to make sure it's installed when your unRaid server is booted or when the Array is started the first time.

#### User scripts

With this option we're going to install the qBit API when the Array is started the first time.

Go to your unRaid Dashboard to your settings tab and select in the `User Utilities` at the bottom the new plugin you installed `User Scripts`.

??? check "Example - [CLICK TO EXPAND]"
    ![!User Scripts](images/Unraid-settings-user-scripts-icon.png)

Select at the bottom `ADD NEW SCRIPT`.

??? check "Example - [CLICK TO EXPAND]"
    ![!Add New Script](images/Unraid-user-scripts-add-new-script-icon.png)

A popup will appear where you can give it a name, for this example we're going to use `Install qBittorrent API` and then click on `OK`.

??? check "Example - [CLICK TO EXPAND]"
    ![!Install qBittorrent API](images/Unraid-user-scripts-add-new-script-enter-name.png)

Click in the list on the cogwheel of the new user scrip you made.

??? check "Example - [CLICK TO EXPAND]"
    ![!Select user script](images/Unraid-settings-user-scripts-list-select-qbit-api.png)

Copy/Paste in the new windows that opens the following bash command followed by `SAVE CHANGES`.

```bash
#!/bin/bash
pip install qbittorrent-api
```

??? check "Example - [CLICK TO EXPAND]"
    ![!Bash script](images/Unraid-settings-user-scripts-qbit-api.png)

Select in the schedule list when the script should run, and choose `At First Array Start Only`.

??? check "Example - [CLICK TO EXPAND]"
    ![!Set Run Time](images/Unraid-settings-user-scripts-qbit-api-schedule.png)

Click on `RUN IN BACKGROUND` or restart your unRaid server so the qBit API is installed.

??? check "Example - [CLICK TO EXPAND]"
    ![!RUN IN BACKGROUND](images/Unraid-settings-user-scripts-qbit-api-run-background.png)

------

#### Go File

With this option we're going to install the qBit API when the unRaid server is started.

On your USB stick/key go to `/boot/config` and open the `go` file in your favorite editor and copy/paste the following command.

```bash
pip install qbittorrent-api
```

Restart your unRaid Server, or run the above command from the terminal.

------

### Script

You only need to edit a few options in the script

```python
# --DEFINE VARIABLES--#
# Set Number of Days to stop torrents for the move
days = 2
qbt_host = '192.168.2.200:8080'
qbt_user = admin
qbt_pass = adminadmin
# --DEFINE VARIABLES--#
```

- `days` => Set Number of Days to stop torrents for the move.
- `qbt_host` => The URL you use to access qBittorrent locally.
- `qbt_user` => Your used qBittorrent `User Name` if you have authentication enabled.
- `qbt_pass` => Your used qBittorrent `Password` if you have authentication enabled.

!!! failure ""
    If you don't use the unRaid `Mover Tuning` app, You might need to change **line 54** from `os.system('/usr/local/sbin/mover.old start')` to `os.system('/usr/local/sbin/mover start')`

#### Copy script to your preferred location

Now it's time to place the script somewhere easy to access/remember.

Suggestions:

- `/mnt/user/appdata/qbittorrent/scripts` (yes you need to create this folder your self)
- `/mnt/user/data/scripts` (yes you need to create this folder your self)

#### Final steps

Now it's time to setup the scheduler when the mover should run.

Go to your unRaid Dashboard to your settings tab and select in the `User Utilities` at the bottom the new plugin you installed `User Scripts`.

??? check "Example - [CLICK TO EXPAND]"
    ![!User Scripts](images/Unraid-settings-user-scripts-icon.png)

Select at the bottom `ADD NEW SCRIPT`.

??? check "Example - [CLICK TO EXPAND]"
    ![!Add New Script](images/Unraid-user-scripts-add-new-script-icon.png)

A popup will appear where you can give it a name, for this example we're going to use `qBittorrent Mover` and then click on `OK`.

??? check "Example - [CLICK TO EXPAND]"
    ![!qBittorrent Mover](images/Unraid-user-scripts-add-new-script-enter-name-qbt.png)

Click in the list on the cogwheel of the new user scrip you made.

??? check "Example - [CLICK TO EXPAND]"
    ![!Select user script](images/Unraid-settings-user-scripts-list-select-qbit-mover.png)

Copy/Paste in the new windows that opens the following bash command followed by `SAVE CHANGES`.

```bash
#!/bin/bash
/usr/local/emhttp/plugins/dynamix/scripts/notify -s "qBittorrent Mover" -d "qBittorrent Mover starting @ `date +%H:%M:%S`."
echo executing script to pause torrents and run mover.
/usr/bin/python3 /mnt/user/data/scripts/mover.py
echo qbittorrent-mover completed and resumed all paused torrents.
/usr/local/emhttp/plugins/dynamix/scripts/notify -s "qBittorrent Mover" -d "qBittorrent Mover completed @ `date +%H:%M:%S`."
```

!!! info
    Replace the `/mnt/user/data/scripts/mover.py` path to the path where you placed your python script.

??? check "Example - [CLICK TO EXPAND]"
    ![!Bash script](images/Unraid-settings-user-scripts-qbit-mover.png)

Select in the schedule list when the script should run, and choose `Custom`

??? check "Example - [CLICK TO EXPAND]"
    ![!Set Run Time](images/Unraid-settings-user-scripts-qbit-mover-schedule.png)

After changing to `Custom` you get on the right a extra option where you can setup your cron schedule when it should be run.

For this example we're going to let the script run a 4am at night. `0 4 * * *`

Setup your own schedule [HERE](https://crontab.guru/)

??? check "Example - [CLICK TO EXPAND]"
    ![!Set Run Time](images/Unraid-settings-user-scripts-qbit-mover-cron.png)

--8<-- "includes/support.md"

[^1]:
    Starr apps = Sonarr/Radarr etc. Doesn't Starr apps sound better then `The arr(s)` ?
