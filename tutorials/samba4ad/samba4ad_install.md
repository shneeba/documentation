# SAMBA4AD
### Prepping the server
Install OS, assign static IP address

Edit host file and add an IP address entry with the hostname and FQDN
```
127.0.0.1   localhost localhost.tobias.local
::1         localhost localhost.tobias.local
10.0.2.201  tobiasdc01 tobiasdc01.tobias.local
```

Run the following to update hostname
```
hostnamectl set-hostname tobiasdc01
```

Reboot

### Installing packages
We need to install the following packages to build Samba as an AD DC
```
yum -y install attr bind-utils docbook-style-xsl gcc gdb krb5-workstation \
       libsemanage-python libxslt perl perl-ExtUtils-MakeMaker \
       perl-Parse-Yapp perl-Test-Base pkgconfig policycoreutils-python \
       python2-crypto gnutls-devel libattr-devel keyutils-libs-devel \
       libacl-devel libaio-devel libblkid-devel libxml2-devel openldap-devel \
       pam-devel popt-devel python-devel readline-devel zlib-devel systemd-devel \
       lmdb-devel jansson-devel gpgme-devel pygpgme libarchive-devel wget ntp
```

Unfortunately the Samba package provided from CentOS official repository does not provide the DC function yet, so we need to download and install from source.

Install the development tools so we can compile and install Samba
```
yum groups -y install "Development Tools"
```

I'm adding the EPL7 repo to download and install the lmdb package to be sure (make sure wget installed)
```
yum install epel-release
yum install lmdb
```

Download the tar ball (double check the samba URL for latest version, [link](https://download.samba.org/pub/samba/stable/)) - I've had to use 4.3.3 as 4.9.3 has an lmdb dependency error
```
wget https://download.samba.org/pub/samba/stable/samba-4.3.3.tar.gz
```

*----- 4.9.3 lmdb dependency fix -----*

Download the lmdb package from github
```
wget https://github.com/LMDB/lmdb/archive/LMDB_0.9.22.tar.gz
```

Extract the tarball
```
tar -zxvf LMDB_0.9.22.tar.gz
```

Change directory into "lmdb-LMDB_0.9.22/libraries/liblmdb/"
```
cd lmdb-LMDB_0.9.22/libraries/liblmdb
```

Run the "make"
```
make
```

Globally remove the "liblmdb.a" reference from the makefile
```
sed -i 's/liblmdb.a//' Makefile
```

Install the lmdb library
```
make prefix=/usr install
```

Now you can continue with the below

*----- 4.9.3 lmdb dependency fix -----*


Extract the tar ball
```
tar -zxvf samba-4.3.3.tar.gz
```

Navigate to the newly created directory
```
cd samba-4.3.3
```

Time to configure the package, run the following (we don't want CUPS)
```
./configure --disable-cups
```

Once the configure has completed we need to start the compilation
```
make
```

Once this is complete we need to install it
```
make install
```

Now it's installed lets add the dirs to the $PATH so we can easily use the tools. Create a "samba.sh" file in "/etc/profile.d/"
```
vi /etc/profile.d/samba.sh
```

Paste the following
```
export PATH=$PATH:/usr/local/samba/bin/:/usr/local/samba/sbin/
```

Save and quit, log off and log back on and make sure you can run "samba-tool".

### Configuring new domain controller
Now Samba is installed we need to do the config side of things.

We probably have some old config in our krb5.conf file, backup this file just in case and link the samba one
```
mv /etc/krb5.conf /etc/krb5.conf.bak
ln -sf /usr/local/samba/private/krb5.conf /etc/krb5.conf
```

We want to now provision the domain, for this example we're using " --option="interfaces=lo enp0s3" --option="bind interfaces only=yes"" to bind to just the interfaces we want.
```
samba-tool domain provision --use-rfc2307 --interactive --option="interfaces=lo enp0s3" --option="bind interfaces only=yes"
```

Once the domain has been configured we need to edit our "resolv.conf" file to point to the new DC like so
```
vi /etc/resolv.conf
```
```
search tobias.local
nameserver 10.0.2.201
```

Create a new "/etc/systemd/system/samba-ad-dc.service" file so we can easily start and stop the service
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

Start samba
```
systemctl start samba-ad-dc
```

Create a reverse DNS record on the new domain controller (remember the password you used when setting up the domain!)
```
samba-tool dns zonecreate 10.0.2.201 2.0.10.in-addr.arpa -Uadministrator
```

### Testing config
Lets query some DNS records to make sure it's working correctly

The tcp-based _ldap SRV record in the domain
```
host -t SRV _ldap._tcp.tobias.local
```

The udp-based _kerberos SRV resource record in the domain
```
host -t SRV _kerberos._udp.tobias.local
```

The A record of the domain controller
```
host -t A tobiasdc01.tobias.local
```

Request a Kerberos ticket for the domain administrator account
```
kinit administrator
```

List the cached Kerberos tickets using "klist"
```
klist
```

This is the initial testing completed 

### Additional config
We need to add a DNS forwarder if we didn't do so in the initial config so that we can talk externally, add the following to your "/usr/local/samba/etc/smb.conf" file:
```
dns forwarder = 8.8.8.8
```

To get this fully working we need to configure NTP, this involves installing and configuring the package. Run the following:
```
yum install ntp
```

Edit /etc/ntp.conf and replace everything with the following (for ease, setting different pool of NTP servers):
```
driftfile /var/lib/ntp/drift
restrict default nomodify notrap nopeer noquery
restrict 127.0.0.1
restrict ::1
server 0.uk.pool.ntp.org
server 1.uk.pool.ntp.org
server 2.uk.pool.ntp.org
server 3.uk.pool.ntp.org
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
disable monitor
```

Add the Samba AD ports to your firewalld config
```
firewall-cmd --add-port=53/tcp --permanent;firewall-cmd --add-port=53/udp --permanent;firewall-cmd --add-port=88/tcp --permanent;firewall-cmd --add-port=88/udp --permanent; \
firewall-cmd --add-port=135/tcp --permanent;firewall-cmd --add-port=137-138/udp --permanent;firewall-cmd --add-port=139/tcp --permanent; \
firewall-cmd --add-port=389/tcp --permanent;firewall-cmd --add-port=389/udp --permanent;firewall-cmd --add-port=445/tcp --permanent; \
firewall-cmd --add-port=464/tcp --permanent;firewall-cmd --add-port=464/udp --permanent;firewall-cmd --add-port=636/tcp --permanent; \
firewall-cmd --add-port=1024-5000/tcp --permanent;firewall-cmd --add-port=3268-3269/tcp --permanent
```

*----- Optional -----*

Disable the firewalld service as this stops communication to your ADDC if you're having issues with firewall ports or firewalld
```
systemctl stop firewalld
```

Check to make sure it's stopped with
```
systemctl status firewalld
```
*----- Optional -----*

This should now allow you to connect computers to the domain and create users along with talking externally. For more information on managing samba via the CLI check out the link [here](https://www.tecmint.com/manage-samba4-active-directory-linux-command-line/).

Join a second domain controller to the domain for redundancy
