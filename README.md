# buttercup

btrfs backup, the successor to `btwrap`.  
Keep in mind this tool is exclusively made for btrfs filesystem(s), backing up to other btrfs filesystem(s).

## Use cases

1. You're running an arch linux system, and new packages might break something; and you want to always have a stable system for all the important things you do on your PC  
   * Given you're also using `grub` as your bootloader, just install the AUR package `buttercup-git` as well as the optional depencency `grub-btrfs`  
   * The package comes with a hook that will use this took to create snapshots, and list them as bootable options in a submenu of `grub`  
   * It will do this just before each linux kernel transaction, meaning each system upgrade that includes your kernel  
   * This makes sure that you always have a functional system to boot to, and it won't take up much space at all usually.  

2. You want to backup your data to any mounted btrfs filesystem, to prevent data loss in case the original drive is unrecoverable  
   * Install `buttercup`, and then you can run `buttercup -o "$origin" -b "$backup"`  
   * with "$origin" being the path of files you want to backup, and "$backup" being the location (preferably empty) that you want to back them up to  
   * The intial backup will be simply copying files over to the backup location, but the beauty of it is that it is very efficient at incrementally updating what it has already written to the backup
   * It will write to subvolumes within "$backup", with their name being in unix time  
  
If you feel so inclined, you can make a couple systemd units to routinely write backups like #2; here's an example:  
```sh
[Unit]
Description='routine backups with buttercup'

[Service]
ExecStart=/usr/bin/buttercup -o "$origin" -b "$backup"

[Install]
WantedBy=multi-user.target
```
Make sure to replace $origin and $backup with the paths on your system  
And then you can write it to '/etc/systemd/system/buttercup.service' for example  

Then, you can create a timer to start the service regularly like so:
```sh
[Unit]
Description='trigger routine backups via buttercup.service'

[Timer]
# trigger the service every week
OnCalendar=weekly
# if the scheduled time was missed, such as because the device was powered off, run the service when it boots up again
Persistent=true

[Install]
WantedBy=timers.target
```
This can be written to '/etc/systemd/system/buttercup.timer'

## The Idea

Tools such as [btrbk](https://github.com/digint/btrbk) can be used for similar purposes, however:  

1. The first thing you always have to do is write a configuration file. `btrbk` can do a lot, which isn't necessarily a good thing.  
   * `buttercup` is made for one purpose, and in contrast doesn't need a configuration file or any data storage of its own to function. It's simply a script that calls `btrfs-progs` in somewhat smarter ways.
2. When you need only a simply task done regularly, a smaller script is much easier to understand and therefore use

In fact, everything `buttercup` does could be done through the shell with the same `btrfs-progs` it uses to function, it just wouldn't be as convinient and leaves too much room for error;
That's why it is a script with a lot of checks to make sure nothing is going wrong, such as preventing itself from deleting an important "parent" snapshot.
