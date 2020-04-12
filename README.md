# Debian repository creation

## Prerequisite

```bash
apt-get update && apt-get install dpkg-sig reprepro -yy
```

## Server side

Generate the passphrase and then store it somewhere.

```bash
date +%s | sha256sum | base64 | head -c 32 ; echo
```
ex : NGU1N2Y2MzXXXXXXXVmZWI3ZWVjZWU4   

Deposit key generation

```bash
$ gpg --gen-key
```

Enter the same name as in the 'Origin' of the repository and your email and then we ask you twice for the passphrase.   
To generate the key, you will have to move the mouse in the terminal afterwards. The terminal will wait until there is enough movement to generate the key encryption which is based on the random movements of the mouse.

```bash
gpg --list-keys
```
Export of public key   
```bash
gpg --output rom.gpg.key --armor --export 'MY_REPO'
```
Copy on that right directory
```bash
cp rom.gpg.key /var/www/repos/apt/debian/key
```
Create all directories of this repo
```bash
mkdir -p /var/www/repos/apt/debian/{conf,db,dists,key,lists,pool,incoming}
```

Create the distribution file
```bash
nano /var/www/repos/apt/debian/conf/distributions
```
```bash
Origin: MY_REPO
Label: RB_REPO
Suite: stable
Codename: stable
Architectures: amd64
Components: main
Description: RB personnal repo
SignWith: yes

Origin: MY_REPO
Label: RB_REPO
Suite: beta
Codename: beta
Architectures: amd64
Components: main
Description: RB personnal repo
SignWith: yes
```

## To include packet on this repo
Include packet on beta repo
```bash
cd /var/www/repos/apt/debian
reprepro includedeb beta /path_of_package/MyPacket.deb
```

## Apache vhost for access in http

```bash
nano /etc/apache2/sites-available/deb.rom.ovh.conf
```
```bash
<VirtualHost *:80>
        ServerName deb.rom.ovh
        DocumentRoot "/var/www/repos/"
        ErrorLog "/var/log/apache2/deb.rom.ovh-errors.log"
        CustomLog /var/log/apache2/deb.rom.ovh-access.log combined

        <Directory "/var/www/repos">
                Options Indexes FollowSymLinks MultiViews
                AllowOverride all
                Order allow,deny
                allow from all
       </Directory>

        <Directory "/var/www/repos/apt/debian/db/">
                Order deny,allow
                Deny from all
        </Directory>

        <Directory "/var/www/repos/apt/debian/conf/">
                Order deny,allow
                Deny from all
        </Directory>

        <Directory "/var/www/repos/apt/debian/incoming/">
                Order allow,deny
                Deny from all
        </Directory>

</VirtualHost>
```  

## Client side
:warning: After the first configuration, you need to add one packet in your repo for sign it. If you don't sign your repo, you will obtain an error on your apt-get update on client side. 

If you don't have gnupg2 packet in your in client, you need to install this.

```bash
apt-get update && apt-get install -y gnupg2
```
```bash
wget -O - http://deb.rom.ovh/apt/debian/key/rom.gpg.key | apt-key add -
echo 'deb http://deb.rom.ovh/apt/debian stable main' >> /etc/apt/sources.list.d/rom.list
```
