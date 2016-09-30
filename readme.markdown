Init Script for Minecraft/Bukkit Servers
========================================
This is a script for starting a single-world vanilla Minecraft server at boot, and comes with a bunch of other features that make maintaining a server fairly simple.

Features
--------
 * Easy interactive setup feature.
 * Help command shows you all commands and their uses.
 * Whitelist commands (on, off, add, remove, and display).
 * Op commands (add, remove, and display).
 * Choose to use a ramdisk to speed up server read/write times to the world files.
 * Specify max RAM the server process should use.
 * Stop, start, restart, and update the server with simple console commands.
 * See who is currently connected, and all connects/disconnects since the last log-roll.
 * Easily push a message to people playing on the server.
 * Use Minecraft server commands without opening the server console screen.
 * Automatic backups using cron jobs.
 * Log-roll function to backup and clean the server.log, as a big logfile can slow down the server.
 * Server updating.
 * Exclude files and directories from full backup by adding them to "exclude.list"


Requirements
============
screen, rsync


Easy Setup
==========
New in version 1.3.0 is the ability to set up a Minecraft server in minutes!

First, install the script by placing it in '/etc/init.d/' as 'minecraft' and run these commands:

		sudo chmod 755 /etc/init.d/minecraft
		sudo update-rc.d minecraft defaults

Then, run:

		sudo service minecraft setup

...and follow the on-screen instructions!

__IMPORTANT:__ After that's done, you'll have to add the cron jobs manually so that things are properly backed up on a schedule. Follow step 7 in "Manual Setup" below.

If you want full control over your setup, see "Manual Setup" below.


Manual Setup
============

1. Symlink the minecraft file to '/etc/init.d/minecraft', set the required premissions and update rc.d.

		sudo ln -s ~/minecraft-init/minecraft /etc/init.d/minecraft
		chmod 755  ~/minecraft-init/minecraft
		sudo update-rc.d minecraft defaults


2. Setup 'minecraft' user and home directory:

		sudo useradd -m minecraft


3. Edit the variables in 'config.example' to your needs and rename it to 'config'

		cp ~/minecraft-init/config.example ~/minecraft-init/config
		nano ~/minecraft-init/config


4. Move your worlds to the $WORLDSTORAGE directory:

		mv <your world directory> /home/minecraft/server/worlds/


5. Setup ramdisk (this script will not run without it):

	Create the ramdisk mount point:

		sudo mkdir /home/minecraft/ramdisk

	Open 'fstab' for editing with:

		sudo nano /etc/fstab

	...and add the line (tab deliniated):

		tmpfs	/home/minecraft/ramdisk	tmpfs	defaults,size=512m	0	0

	Here, I'm setting my ramdisk up with 512MB, which is more than enough for my 43MB total world files; you may wish to use more or less. _Make sure you have enough RAM for this!_


	The ramdisk will mount every time you boot. For the first time you have to mount it manually:

		sudo mount /home/minecraft/ramdisk

	See http://www.minecraftwiki.net/wiki/Tutorials/Ramdisk_enabled_server for more info on ramdisks.


	To load a world from ramdisk run:

		/etc/init.d/minecraft ramdisk WORLDNAME

	to disable ramdisk, run the same command again.


6. If you haven't already, download minecraft_server.jar to $MCPATH (or move it if you are applying this script to an existing server).

		cd /home/minecraft/server
		sudo wget 'https://s3.amazonaws.com/MinecraftDownload/launcher/minecraft_server.jar'


7. Make sure permissions on ALL files and folders in /home/minecraft are owned by the 'minecraft' user:

		cd /home
		sudo chown -R minecraft:minecraft minecraft

	You can check permissions on all files within a folder by 'cd''ing into it and using:

		ls -al


8. Edit 'crontab' to do a backup at 5am and copy the world from the ramdisk every 10 minutes:

		sudo crontab -e

	Add these lines (tab deliniated):

		#m 	h 	dom	mon	dow	command
		05 	05 	*	*	*	/etc/init.d/minecraft backup
		00 	05 	*	*	*	/etc/init.d/minecraft log-roll
		*/10 	* 	*	*	*	/etc/init.d/minecraft to-disk


And that _should_ be all there is to it. Please file an Issue if you have trouble with this setup.

For more help with the script, run
	/etc/init.d/minecraft help


Usage
=====
The script starts your server at boot and, if you added the cron jobs, will backup your worlds every day.

You can issue a command to the script by calling:

		sudo service minecraft [COMMAND]

__Available commands:__
 * setup				Interactive prompt for setting up the server with this script
 * start				Starts the server
 * stop					Stops the server
 * restart				Restarts the server
 * backup				Backups the worlds defined in the script
 * full-backup			Backups the entire server folder
 * to-disk				Copies the world from the ramdisk to world_storage
 * update				Fetches the latest version of $SERVICE
 * log-roll				Moves and gzips the logfile
 * say					Prints the given string to the in-game chat
 * command				Executes a command in-game
 * connected			Lists connected users
 * recent				Displays recently connected users
 * whitelist			Displays server whitelist
 * whitelist-on			Tells server to use whitelist (server open to only those players listed in whitelist).
 * whitelist-off		Tells server to NOT use whitelist (server open to all players).
 * whitelist-add NAME	Adds the specified player to the server whitelist
 * whitelist-remove NAME	Removes the specified player from the server whitelist
 * ops					Displays server ops
 * op NAME				Grants NAME operator status.
 * deop NAME			Revokes NAME's operator status.
 * console				Displays the server console screen, exit with Ctrl+A, D
 * status				Displays server status
 * version				Displays $SERVICE version and then exits
 * fix-permissions		Sets ownership of all files in /home/$USERNAME to $USERNAME
 * help					Displays this list of commands


Access server console
=====================
	screen -r minecraft

Exit the console
	Ctrl+A D


To Do List
==========
 * Update updater for 1.6.2+
 * Autodetect WORLDNAME
 * Provide commands for restoring backups
 * Remove backups older than a specified time period
 * Add/remove from whitelist/ops when server is not running
 * Automatically detect errors in java process and restart accordingly
 * Only perform world backup if change has been detected (players logged in, level.dat change in size, etc)


'Tested on Ubuntu 12.04.1 server; may work on other flavors of Debian'
