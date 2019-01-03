# Joining Centos7 server to domain

### Prepping the server

_Guide followed [here](https://www.tecmint.com/integrate-centos-7-to-samba4-active-directory/)_

Install OS, assign static IP address

Configure local DNS (/etc/resolv.conf) to point to the new AD DC
```
search tobias.local
nameserver 10.0.2.201
```

Edit host file and add an IP address entry with the hostname and FQDN
```
127.0.0.1   localhost localhost.tobias.local
::1         localhost localhost.tobias.local
10.0.2.203  tobiasrh01 tobiasdc01.tobias.local
```

Run the following to update hostname
```
hostnamectl set-hostname tobiasdc01
```

Reboot

Install and configure NTP to sync with the domain controller
```
yum install ntp
ntpdate tobias.local
```

Install the required packages
```
yum -y install authconfig samba-winbind samba-client samba-winbind-clients
```

We can use a nice little GUI to join the domain
```
authconfig-tui
```

At the first prompt screen make sure the following are selected
```
Use Winbind
Use Shadow Password
Use Winbind Authentication
Local authorization is sufficient
```

On the second prompt screen fill in as following
```
Security Model: ads
Domain = TOBIAS
Domain Controllers = tobiasdc01.tobias.local,tobiasdc02.tobias.local
ADS Realm = TOBIAS.LOCAL
Template Shell = /bin/bash
```

_----- Optional -----_

We can join the domain without a GUI utility, however this can be prone to typos ect
```
authconfig --enablewinbind --enablewinbindauth --smbsecurity ads --smbworkgroup=TOBIAS --smbrealm TOBIAS.LOCAL --smbservers=tobiasdc01.tobias.local,tobiasdc02.tobias.local --krb5realm=TOBIAS.LOCAL --enablewinbindoffline --enablewinbindkrb5 --winbindtemplateshell=/bin/bash--winbindjoin=Administrator --update  --enablelocauthorize   --savebackup=/backups
```

_----- Optional -----_

### Testing and verifying the config

We can verify if the winbind service is running by using the following command
```
systemctl status winbind.service
```

Edit the local "smb.conf" found in "/etc/samba/smb.conf" and make sure the following are set at the end of the "[global]" configuration
```
winbind use default domain = true
winbind offline logon = true
```

To make sure local home directories are created run the following
```
authconfig --enablemkhomedir --update
```

Finally restart winbind
```
systemctl restart winbind
```

Log off the server and then try and log back in with a domain user.
