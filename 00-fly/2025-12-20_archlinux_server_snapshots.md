# Snapshots (BTRFS)

Despite perhaps not being conventional or usual, I've chosen btrfs as the underlying FS for my VPS the reason is cos this will be running virtual machines and/or containers, in case of VMs I will be using other FS (ie: zfs) but as I'm using archlinux and I won't be ignoring packages, I want to have a safety network.

the idea is:

* Set up an update calendar (ie: update every 1 or 2 weeks)
* No ignoring packages: ignoring pacakages it could be a good idea to have a more robust and stable system (ignore a package means that it won't updated on `sudo pacman -Syyu` despite having a newer version). However, ignoring could also break the system if some dependencies are updated while the dependant packages are not (ie: if a library introduce a breaking change and is updated, the dependant packages will most likely break).
* Configure btrfs, timeshift and grub to perform automatic snapshots and being able to restore them from grub. For the time being the snapshots will live in the same disk as the main system, this is perhaps not the best practise but this is a home lab and I don't have available at the moment other disks to save those snapshots, in the future we will be saving the snapshots to remote systems.

As pre-conditions, we need to have a BTRFS system with the usual subvolumes:

* @     -> / (root)
* @home -> /home/<user>
* @log  -> /var/log
* @pkg  -> /var/cache/pacman/pkg

so, let's begin.

## Setting up timeshift

Let's see first the timeshift configuration:

1. Install: `sudo pacman -S timeshift`
2. Produce timeshift configuration for btrfs: `sudo timeshift --btrfs` this command will produce the configuration at: `/etc/timeshift/timeshift.json`
3. Find the UUID of the device to take snapshots, to do this run: `sudo blkid`
4. Configure timshift to take snapshots from that particular UUID, ie:

```
{
  "backup_device_uuid" : "<UUID>"
  "parent_device_uuid" : "<UUID>"
  "do_first_run" : "false",
  "btrfs_mode" : "true",
  "include_btrfs_home_for_backup" : "false",
  "include_btrfs_home_for_restore" : "false",
  "stop_cron_emails" : "true",
  "schedule_monthly" : "true",
  "schedule_weekly" : "true",
  "schedule_daily" : "true",
  "schedule_hourly" : "false",
  "schedule_boot" : "false",
  "count_monthly" : "1",
  "count_weekly" : "1",
  "count_daily" : "2",
  "count_hourly" : "6",
  "count_boot" : "5",
  "date_format" : "%Y-%m-%d %H:%M:%S",
  "exclude" : [],
  "exclude-apps" : []
}
```

5. Save the file

## Setting up GRUB

We will configure GRUB to be able to restore snapshots.

1. Install grub-btrfs: `sudo pacman -S grub-btrfs`
2. Enable the service: `sudo systemctl enable --now grub-btrfsd` (note teh D for daemon at the end)
3. (It should be done but just in case) recreate the grub config: `sudo grub-mkconfig -o /boot/grub/grub.cfg`
4. Check the status: `sudo systemctl status grub-btrfsd` 

It hosuld be up and running.

## Setting up autosnap

timshift-autosnap is a small script that will create a nsapshot every time you update packages, assuming that you are using `yay` as AUR manager, then:

1. Install it `yay -S timeshift-autosnap` 
2. You can test it with: `sudo timeshift-autosnap` or just `sudo pacman -S <some package>`

