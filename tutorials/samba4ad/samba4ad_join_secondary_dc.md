# Join a second domain controller to the domain for redundancy
**_Follow the previous guide up until the "Configuring new domain controller" section._**

### Prepping the secondary server

Configure local DNS (/etc/resolv.conf) to point to the new AD DC
```
search tobias.local
nameserver 10.0.2.201
```

Create a new "/etc/systemd/system/samba-ad-dc.service" file
```
vi /etc/systemd/system/samba-ad-dc.service
```

Paste the following in
```
[Unit]
Description=Samba Active Directory Domain Controller
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
ExecStart=/usr/local/samba/sbin/samba -D
PIDFile=/usr/local/samba/var/run/samba.pid
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```

Reload "systemd" config
```
systemctl daemon-reload
```

Enable / disable samba AD on startup
```
systemctl enable samba-ad-dc
systemctl disable samba-ad-dc
```

We can now restart samba-ad-dc with the following commands
```
systemctl start samba-ad-dc
systemctl stop samba-ad-dc
```

Make a backup of your "/etc/krb5.conf" file
```
mv /etc/krb5.conf /etc/krb5.conf.bak
```

Create a new /etc/krb5.conf.bak file with the following
```
[libdefaults]
        default_realm = TOBIAS.LOCAL
        dns_lookup_realm = false
        dns_lookup_kdc = true
```

Verify the new Kerberos settings work and request a Kerberos ticket for the domain administrator
```
kinit administrator
```

List Kerberos tickets to be sure
```
klist
```

### Joining secondary DC to domain

Run the below command to join the domain using the domain administrator user
```
samba-tool domain join tobias.local DC -U"TOBIAS\administrator" --dns-backend=SAMBA_INTERNAL --option="interfaces=lo enp0s3" --option="bind interfaces only=yes" --option='idmap_ldb:use rfc2307 = yes'
```

I was having some DNS issues finding local servers after joining the domain, I restarted the samba service on each DC in turn and everything looks to work as normal now (internal and external DNS), I think this was partly due to how I went through the whole process before refining.

