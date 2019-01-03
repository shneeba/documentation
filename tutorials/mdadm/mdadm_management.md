# Managing mdadm

### Add and remove devices

To add a device
```
mdadm --add /dev/md0 /dev/sdd
```

To remove a device (-f fail device, -r remove device)
```
# mdadm --manage /dev/md0 -f /dev/sdd
# mdadm --manage /dev/md0 -r /dev/sdd
```

### Replacing failed mirror disk ([link](https://www.thegeekdiary.com/replacing-a-failed-mirror-disk-in-a-software-raid-array-mdadm/))

Before removing raid disks we need to run the following command to write all disk caches to the disk
```
sync    
```

Mark as failed
```
mdadm --manage /dev/md0 --fail /dev/sdb1
```

Check /proc/mdstat to make sure the disk has been marked as failed (F)
```
# cat /proc/mdstat

Personalities : [linear] [multipath] [raid0] [raid1] [raid5] [raid4] [raid6] [raid10]

md0 : active raid1 sda1[0] sdb1[2]**(F)**

      976773168 blocks [2/1] [U_]

The \*\*(F)\*\* shows the disk has failed

We can now physically remove the disk and replace
```
_Remove and replace disk_
```

We now need to copy the partition table to the newly replaced disk
```
sfdisk -d /dev/sda | sfdisk /dev/sdb
```

Create a mirror of the disk
```
mdadm /dev/md0 --add /dev/sdb1
```

Test the setup
```
mdadm --detail /dev/md0
```

Finally check the progress of the recovery
```
cat /proc/mdstat
```
