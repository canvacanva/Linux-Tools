# VM Linux first Steps

Primi passi preparazione macchina virtuale linux

## Domain Controller

Registrare la entry DNS su domain controller.

## Aggiornare tutti i pacchetti
```
sudo su
```
```
sudo apt-get update && sudo apt-get upgrade -y
```

## Open VM-Tools
Aggiornare o installare TOOLS per Vm
```
sudo apt-get install open-vm-tools
```

## Add user
```
sudo adduser userNameHere
sudo usermod -aG sudo [name-of-user]
```

## NTP
```
sudo apt-get install ntp -y
```
Se necessario scegliere un timezione server diverso

```
sudo timedatectl set-timezone Europe/Rome
sudo echo "#Custom NTP Server" >> /etc/ntp.conf
sudo echo "pool ntp.dominiocliente.estensione" >> /etc/ntp.conf
```
```
sudo service ntp stop
sudo ntpd -gq
sudo service ntp start
```
Verificare se l'orario risulta corretto:
```
uptime
```

## CIFS

Necessario per monatre cartelle di rete
```
apt-get install cifs-utils
```

## LOGS

Montare la cartella dei Logs DFS (nel cado di una macchina docker)


Creo cartella locale
```
mkdir /mnt/logs
```

Edito /etc/fstab
```
//istprage/LOGS_APPS/ /mnt/logs cifs uid=0,credentials=/root/.smbcred-logs,iocharset=utf8,vers=2.0,noperm 0 0
```

Edito /root/.smbcred-logs
```
username=svc-user
password=
domain=Domain
```

Rimonto le cartelle
```
mount -a
```

## User Group
user in group www-data: ```grep ^www-data /etc/group```
add needed user: ```usermod -a -G www-data USEEEER```


## Reboot

Riavvio di domenica delle Vm Linux da crontab di sudo
```
0 14 * * 0 /sbin/reboot
```

## CERIFICATO CA INTERNO
Copio i Certificati direttamente dalla macchina CASUB (o li copio in modo alternativo per altre CA)

```
mkdir /usr/share/ca-certificates/company
cd /usr/share/ca-certificates/comapany
wget http://linuxca.corp/RootCaDer.crt
wget http://linuxca.corp/SubCaDer.crt
echo "company/RootCaDer.crt" >> /etc/ca-certificates.conf
echo "company/SubCaDer.crt" >> /etc/ca-certificates.conf
```

Verificare che venga richiesta la nuova CA (comando ASK)
```
dpkg-reconfigure ca-certificates
update-ca-certificates
```

Test del certificato
```
openssl s_client -showcerts -connect SERVIZIO:PORTA
```
## AGGIORNAMENTO CERTIFICATI USER_TRUST

Creare lo script per l'aggiormento ca
```
mkdir /scripts
nano /scripts/update_ca.sh
chmod + x /scripts/update_ca.sh
```

```
#!/bin/sh
cd /tmp/
wget https://gogetssl-cdn.s3.eu-central-1.amazonaws.com/wiki/gogetssl-rsa-dv-ca.crt
wget https://gogetssl-cdn.s3.eu-central-1.amazonaws.com/wiki/AAA_certificate_RSA.crt
mv *.crt /etc/ssl/certs/
c_rehash /etc/ssl/certs/
exit
```

Modificare Crontab di sudo
```
00 23 * * 0 /scripts/update_ca.sh
```

# VM Linux - Enlarge Partitions

Allargare partizioni VM linux

Verificare l'attuale capacità
```
df -h
```

## Scansione di modifiche ai dischi
```
sudo su
```
Sostituire X con la partizione da allargare (solitamente sda)
```
echo 1>/sys/class/block/sdX/device/rescan
```
### Utilizzo di Parted
Avvio parted
```
parted
```
Visualizzare partizione
```
print
```
Scegliere la partizione da allargare (solitamente parteizione 3)
```
resizepart (numero partizione)
```
```
quit
```

### Estedere partizione
Scegliere la partizione da allargare
```
pvresize /dev/sdaX
```
```
lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
```
### Verificare la nuova capacità
```
df -h
```


# IBM As400 connect su Linux
https://ibmi-oss-docs.readthedocs.io/en/latest/odbc/installation.html
```
sudo apt update
curl https://public.dhe.ibm.com/software/ibmi/products/odbc/debs/dists/1.1.0/ibmi-acs-1.1.0.list | sudo tee /etc/apt/sources.list.d/ibmi-acs-1.1.0.list
sudo apt update
sudo apt install ibm-iaccess
```

Creo connessione SQL
```
[DataTest]
Description             = IBM i Access ODBC Driver 64-bit
Driver                  = IBM i Access ODBC Driver 64-bit
System                  = 192.168.....
UserID                  = PSW
Password                = PSW
Naming                  = 0
DefaultLibraries        = Libreria di Default
Database                =
ConnectionType          = 0
CommitMode              = 2
ExtendedDynamic         = 1
DefaultPkgLibrary       = QGPL
DefaultPackage          = A/DEFAULT(IBM),2,0,1,0,512
AllowDataCompression    = 1
MaxFieldLength          = 32
BlockFetch              = 1
BlockSizeKB             = 128
ExtendedColInfo         = 0
LibraryView             = ENU
AllowUnsupportedChar    = 0
ForceTranslation        = 0
Trace                   = 0
```

Connetto
```
isql DataTest
```

# SQL ODBC DIVER su Linux
https://learn.microsoft.com/it-it/sql/connect/php/installation-tutorial-linux-mac?view=sql-server-ver16#installing-on-ubuntu

Esempio versione di PHP precedenti:
```
sudo wget https://packages.microsoft.com/keys/microsoft.asc | sudo tee /etc/apt/trusted.gpg.d/microsoft.asc
sudo curl https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/prod.list > /etc/apt/sources.list.d/mssql-release.list
# aggiungere trust sulla repo scaricata: #add deb [trusted=yes]
nano /etc/apt/sources.list.d/mssql-release.list
#deb [trusted=yes] https://packages.microsoft.com/ubuntu/22.04/prod jammy main
sudo apt-get update
sudo ACCEPT_EULA=Y apt-get install -y msodbcsql17 /oppure sudo ACCEPT_EULA=Y apt-get install -y msodbcsql18
sudo ACCEPT_EULA=Y apt-get install -y mssql-tools
apt-get install -y unixodbc-dev
sudo apt-get install php7.4-dev
sudo pecl install sqlsrv /oppure sudo pecl install sqlsrv-5.10.1
sudo pecl install pdo_sqlsrv /oppure sudo pecl install pdo_sqlsrv-5.10.1
printf "; priority=20\nextension=sqlsrv.so\n" > /etc/php/8.1/mods-available/sqlsrv.ini //usare la versine php corretta
printf "; priority=30\nextension=pdo_sqlsrv.so\n" > /etc/php/8.1/mods-available/pdo_sqlsrv.ini //usare la versine php corretta
exit
sudo phpenmod -v 7.4 sqlsrv pdo_sqlsrv
# Continuare poi con la guida su link editanto di conseguenza
```

