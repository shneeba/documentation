# mdadm
### _Configuring Software raid ([link](https://www.thegeekdiary.com/redhat-centos-managing-software-raid-with-mdadm/))_

We need to make sure all disks have the same partition layout:
```
sfdisk -d /dev/sdb | sfdisk /dev/sdc
```

Now create the RAID device using mdadm
```
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
```

Format the drive and mount it (dir will need to exist)
```
mke2fs -j /dev/md0 && mount /dev/md0 /data01
```

Add fstab entry so persists across reboots
```
/dev/md0       /data01         ext4    defaults        0    0
```

### Verifying the config

/proc/mdstat is a file maintained by the kernel which contains the real time information about the RAID arrays and devices.
```
cat /proc/mdstat
```

If you want some more detailed info use mdadm
```
mdadm --detail /dev/md0
```
