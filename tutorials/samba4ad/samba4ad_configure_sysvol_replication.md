# Configuring SysVol replication

Configure SSH keys so we can rsync automatically without any user interference. _**Do not use a passphrase**_

Create the key on the _primary_ domain controller
```
ssh-keygen -t RSA
```

Copy the key to the secondary domain controller
```
ssh-copy-id root@tobiasdc02
```

Just SSH in to make sure it's working correctly
```
ssh tobiasdc02
```

Once we're in and confirmed the key is working we can log back out.
```
exit
```

On our primary domain controller we can run the below command as a dry run to make sure that we're able to copy the sysvol folder. We're using the -XAavz flags (Xtended attributes, Acls, archive, verbose, z(compress))
```
rsync --dry-run -XAavz --chmod=775 --delete-after  --progress --stats  /usr/local/samba/var/locks/sysvol/ root@tobiasdc02:/usr/local/samba/var/locks/sysvol/
```

If this looks correct then remove the --dry-run part
```
rsync -XAavz --chmod=775 --delete-after  --progress --stats  /usr/local/samba/var/locks/sysvol/ root@tobiasdc02:/usr/local/samba/var/locks/sysvol/
```

Finally edit the crontab
```
crontab -e
```

Paste in the following to sync every 5 minutes
```
*/5 * * * * rsync -XAavz --chmod=775 --delete-after  --progress --stats  /usr/local/samba/var/locks/sysvol/ root@tobiasdc02:/usr/local/samba/var/locks/sysvol/ &gt; /var/log/sysvol-replication.log 2&gt;&1
```

