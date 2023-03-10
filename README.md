# repoverse

Research Data Repository

## Linux

Using Debian stable Bulleye 64

```
mgarcia@mordor:~/Work/repoverse$ vagrant init debian/bullseye64

```

## Java

Installing the suggested version of Java, version 11:

```
vagrant@repoverse:~$ sudo apt install openjdk-11-jdk
```

Maybe we can use a newer version like 17 or 18 (or even 20). But first, check if Java is working

```
vagrant@repoverse:~$ sudo update-alternatives --list java
/usr/lib/jvm/java-11-openjdk-amd64/bin/java
vagrant@repoverse:~$ java -version
openjdk version "11.0.18" 2023-01-17
OpenJDK Runtime Environment (build 11.0.18+10-post-Debian-1deb11u1)
OpenJDK 64-Bit Server VM (build 11.0.18+10-post-Debian-1deb11u1, mixed mode, sharing)
vagrant@repoverse:~$
```

## Payara

Using [Payara](https://www.payara.fish/downloads/payara-platform-community-edition/) version 5 as on Dataverse documentation. After downloading to the host machine, I uploaded to the VM:

```
mgarcia@mordor:~/Work/repoverse$ vagrant upload ~/Downloads/payara-5.2022.5.zip
Uploading /home/mgarcia/Downloads/payara-5.2022.5.zip to payara-5.2022.5.zip
Upload has completed successfully!

  Source: /home/mgarcia/Downloads/payara-5.2022.5.zip
  Destination: payara-5.2022.5.zip
mgarcia@mordor:~/Work/repoverse$
```

And after the upload

```
vagrant@repoverse:~$ ls
payara-5.2022.5.zip
vagrant@repoverse:~$
```

Installed `unzip`

```
vagrant@repoverse:~$ sudo apt install unzip
```

Extrancting `payara`, and moving it to `/usr/local/`

```
vagrant@repoverse:~$ unzip payara-5.2022.5.zip
Archive:  payara-5.2022.5.zip
   creating: payara5/
   (...)
vagrant@repoverse:~$ sudo mv payara5 /usr/local/
vagrant@repoverse:~$ ls /usr/local/
bin  etc  games  include  lib  man  payara5  sbin  share  src
vagrant@repoverse:~$
vagrant@repoverse:~$ ll /usr/local
total 36K
(...)
drwxr-xr-x 7 vagrant vagrant 4.0K Dec  9 12:37 payara5
```

Added the user `payara` to own the Dataverse related software

```
vagrant@repoverse:~$ sudo useradd dataverse
```

Changing ownership of some directories in `payara5` to `dataverse` user

```
vagrant@repoverse:~$ sudo chown -R root:root /usr/local/payara5
vagrant@repoverse:~$ sudo chown dataverse /usr/local/payara5/glassfish/lib
vagrant@repoverse:~$ sudo chown -R dataverse:dataverse /usr/local/payara5/glassfish/domains/domain1
```

It seems that it's not necessary to change in the `domain.xml` since it already has `-server` (instead of `-client` like in previous version of Glassfish)

```
vagrant@repoverse:~$ grep  'jvm-options' /usr/local/payara5/glassfish/domains/domain1/config/domain.xml | grep '-server' | head -n 1
                <jvm-options>-server</jvm-options>
vagrant@repoverse:~$
```

### Launching Payara on System Boot

For the moment, I'm skipping this step since the documentation states that Dataverse will start Payara as needed. The docuementation also provides a service script if needed. For the moment I just downloaded the script:

```
vagrant@repoverse:~$ wget https://guides.dataverse.org/en/latest/_downloads/c08a166c96044c52a1a470cc2ff60444/payara.service
vagrant@repoverse:~$ head -n 5 payara.service
[Unit]
Description = Payara Server
After = syslog.target network.target
(...)
```

Copying the script to systemd directory, but not start at boot time. I will wait for the configuration of dataverse itself, to decide if it's necessary to start before or if we can wait for dataverse to start payara.

```
vagrant@repoverse:~$ sudo cp payara.service /etc/systemd/system/
vagrant@repoverse:~$
vagrant@repoverse:~$ sudo systemctl daemon-reload
vagrant@repoverse:~$
vagrant@repoverse:~$ sudo systemctl start payara
vagrant@repoverse:~$
vagrant@repoverse:~$ sudo systemctl status payara
● payara.service - Payara Server
     Loaded: loaded (/etc/systemd/system/payara.service; disabled; vendor preset: enabled)
     Active: active (running) since Sat 2023-03-04 18:47:50 UTC; 9s ago
    Process: 1968 ExecStart=/usr/bin/java -jar /usr/local/payara5/glassfish/lib/client/appserver-cli.jar start-domain (code=exited, status=0/SUCCES>
   Main PID: 2006 (java)
      Tasks: 153 (limit: 9505)
     Memory: 632.1M
        CPU: 28.109s
     CGroup: /system.slice/payara.service
             └─2006 /usr/lib/jvm/java-11-openjdk-amd64/bin/java -cp /usr/local/payara5/glassfish/domains/domain1/lib/ext/*:/usr/local/payara5/glass>

Mar 04 18:47:41 repoverse systemd[1]: Starting Payara Server...
Mar 04 18:47:49 repoverse java[1968]: Waiting for domain1 to start ........
Mar 04 18:47:49 repoverse java[1968]: Successfully started the domain : domain1
Mar 04 18:47:49 repoverse java[1968]: domain  Location: /usr/local/payara5/glassfish/domains/domain1
Mar 04 18:47:49 repoverse java[1968]: Log File: /usr/local/payara5/glassfish/domains/domain1/logs/server.log
Mar 04 18:47:49 repoverse java[1968]: Admin Port: 4848
Mar 04 18:47:49 repoverse java[1968]: Command start-domain executed successfully.
Mar 04 18:47:50 repoverse systemd[1]: Started Payara Server.
vagrant@repoverse:~$
```

## PostgreSQL

### Installiing PostgreSQL

Installing `postgres` from the Debian packages

```
vagrant@repoverse:~$ sudo apt install postgresql
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libpq5 libxslt1.1 postgresql-13 postgresql-client-13 postgresql-client-common postgresql-common ssl-cert sysstat
Suggested packages:
  postgresql-doc postgresql-doc-13 libjson-perl isag
The following NEW packages will be installed:
  libpq5 libxslt1.1 postgresql postgresql-13 postgresql-client-13 postgresql-client-common postgresql-common ssl-cert sysstat
0 upgraded, 9 newly installed, 0 to remove and 0 not upgraded.
Need to get 18.1 MB of archives.
After this operation, 60.2 MB of additional disk space will be used.
Do you want to continue? [Y/n]
Get:1 https://deb.debian.org/debian bullseye/main amd64 libpq5 amd64 13.9-0+deb11u1 [180 kB]
Get:2 https://deb.debian.org/debian bullseye/main amd64 libxslt1.1 amd64 1.1.34-4+deb11u1 [240 kB]
Get:3 https://deb.debian.org/debian bullseye/main amd64 postgresql-client-common all 225 [89.3 kB]
Get:4 https://deb.debian.org/debian bullseye/main amd64 postgresql-client-13 amd64 13.9-0+deb11u1 [1508 kB]
Get:5 https://deb.debian.org/debian bullseye/main amd64 ssl-cert all 1.1.0+nmu1 [21.0 kB]
Get:6 https://deb.debian.org/debian bullseye/main amd64 postgresql-common all 225 [237 kB]
Get:7 https://deb.debian.org/debian bullseye/main amd64 postgresql-13 amd64 13.9-0+deb11u1 [15.2 MB]
Get:8 https://deb.debian.org/debian bullseye/main amd64 postgresql all 13+225 [64.7 kB]
Get:9 https://deb.debian.org/debian bullseye/main amd64 sysstat amd64 12.5.2-2 [603 kB]
Fetched 18.1 MB in 5s (3404 kB/s)
(...)
```

It wasn't clear to me if the service is running or not

```
# Using the `pgcluster` or `systemctl` gave the same result.
vagrant@repoverse:~$ sudo pg_ctlcluster 13 main start
vagrant@repoverse:~$ systemctl status postgresql
● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Fri 2023-02-24 10:06:57 UTC; 8min ago
   Main PID: 2355 (code=exited, status=0/SUCCESS)
      Tasks: 0 (limit: 9505)
     Memory: 0B
        CPU: 0
     CGroup: /system.slice/postgresql.service
vagrant@repoverse:~$
vagrant@repoverse:~$ sudo pg_ctlcluster 13 main stop
vagrant@repoverse:~$
vagrant@repoverse:~$
vagrant@repoverse:~$ systemctl status postgresql
● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Fri 2023-02-24 10:06:57 UTC; 9min ago
   Main PID: 2355 (code=exited, status=0/SUCCESS)
      Tasks: 0 (limit: 9505)
     Memory: 0B
        CPU: 0
     CGroup: /system.slice/postgresql.service
vagrant@repoverse:~$ sudo systemctl start postgresql
vagrant@repoverse:~$
vagrant@repoverse:~$ systemctl status postgresql
● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Fri 2023-02-24 10:06:57 UTC; 9min ago
   Main PID: 2355 (code=exited, status=0/SUCCESS)
      Tasks: 0 (limit: 9505)
     Memory: 0B
        CPU: 0
     CGroup: /system.slice/postgresql.service
vagrant@repoverse:~$
```

### Configuring the Database

The Postgresql configuration using `pg_conftool`

```
vagrant@repoverse:~$ pg_conftool -s 13 main show all
cluster_name = '13/main'
data_directory = '/var/lib/postgresql/13/main'
datestyle = 'iso, mdy'
default_text_search_config = 'pg_catalog.english'
dynamic_shared_memory_type = posix
external_pid_file = '/var/run/postgresql/13-main.pid'
hba_file = '/etc/postgresql/13/main/pg_hba.conf'
ident_file = '/etc/postgresql/13/main/pg_ident.conf'
lc_messages = 'C.UTF-8'
lc_monetary = 'C.UTF-8'
lc_numeric = 'C.UTF-8'
lc_time = 'C.UTF-8'
log_line_prefix = '%m [%p] %q%u@%d '
log_timezone = 'Etc/UTC'
max_connections = 100
max_wal_size = 1GB
min_wal_size = 80MB
port = 5432
shared_buffers = 128MB
ssl = on
ssl_cert_file = '/etc/ssl/certs/ssl-cert-snakeoil.pem'
ssl_key_file = '/etc/ssl/private/ssl-cert-snakeoil.key'
stats_temp_directory = '/var/run/postgresql/13-main.pg_stat_tmp'
timezone = 'Etc/UTC'
unix_socket_directories = '/var/run/postgresql'
vagrant@repoverse:~$
```

Or to show just the `pg_hba.conf` file

```
vagrant@repoverse:~$ pg_conftool -s 13 main show hba_file
/etc/postgresql/13/main/pg_hba.conf
vagrant@repoverse:~$
```

No need to edit `pg_hba.conf` since already has the correct configuration for PostgreSQL and Payara running on the same host

```
vagrant@repoverse:~$ sudo grep -v '^#' /etc/postgresql/13/main/pg_hba.conf | uniq

local   all             postgres                                peer

local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
```

[Accessing](https://www.linuxcapable.com/how-to-install-postgresql-on-debian-11-bullseye/) the PostgreSQL server

```
vagrant@repoverse:~$ sudo -i -u postgres
postgres@repoverse:~$
postgres@repoverse:~$ psql
psql (13.9 (Debian 13.9-0+deb11u1))
Type "help" for help.

postgres=#
```

Setting the [password for user `postgres`](https://stackoverflow.com/questions/27107557/what-is-the-default-password-for-postgres) so later we can provide it to the Dataverse installer

```
vagrant@repoverse:~$ sudo -i -u postgres
postgres@repoverse:~$
postgres@repoverse:~$ psql
psql (13.9 (Debian 13.9-0+deb11u1))
Type "help" for help.

postgres=#
postgres=# ALTER USER postgres PASSWORD 'mynewpasswd';
ALTER ROLE
postgres=#
postgres=# exit
postgres@repoverse:~$ exit
logout
vagrant@repoverse:~$
```

We are going to enable connections from the network to Postgres. Although it seems that this is only necessary when Payara and Postgres are on different server, but we will set up in case one day we change the configuration, and, then, we'll not need to worry about this configuration detail

```
vagrant@repoverse:~$ sudo vim /etc/postgresql/13/main/postgresql.conf
vagrant@repoverse:~$
vagrant@repoverse:~$ diff /etc/postgresql/13/main/postgresql.conf \
> /etc/postgresql/13/main/postgresql.conf_orig
60c60
< listen_addresses = '*'                        # what IP address(es) to listen on;
---
> #listen_addresses = 'localhost'               # what IP address(es) to listen on;
vagrant@repoverse:~$
```

Finally we restart the server

```
vagrant@repoverse:~$ sudo systemctl restart postgresql
```

## Apache Solr

Installing the supported version of Solr, Solr 8.x. We download from Apache Solr website the tarball

```
vagrant@repoverse:~$ wget -O solr-8.11.2.tgz https://www.apache.org/dyn/closer.lua/lucene/solr/8.11.2/solr-8.11.2.tgz?action=download
```

Next we extract the files

```
vagrant@repoverse:~$ sudo mkdir /usr/local/solr
vagrant@repoverse:~$ sudo tar zxf solr-8.11.2.tgz -C /usr/local/solr
vagrant@repoverse:~$ ls /usr/local/solr
solr-8.11.2
vagrant@repoverse:~$
```

Then we change the ownership of the directory

```
vagrant@repoverse:~$ sudo chown -R solr:solr /usr/local/solr
```

finally we configure Solr

```
vagrant@repoverse:~$ sudo -i -u solr
sudo: unable to change directory to /home/solr: No such file or directory
$ cd /usr/local/solr/
$ ls
solr-8.11.2
$ cd solr-8.11.2
$ ls
CHANGES.txt  LICENSE.txt  LUCENE_CHANGES.txt  NOTICE.txt  README.txt  bin  contrib  dist  docs  example  licenses  server
$
$ cp -r server/solr/configsets/_default server/solr/collection1
$
```

Downloading and extracting Dataverse package files

```
vagrant@repoverse:~$ wget https://github.com/IQSS/dataverse/releases/download/v5.13/dvinstall.zip
vagrant@repoverse:~$ unzip dvinstall.zip -d /tmp/
Archive:  dvinstall.zip
  inflating: /tmp/dvinstall/as-setup.sh
  inflating: /tmp/dvinstall/dataverse.war
  (...)
```

Copy Dataverse configuration files to Solr directory

```
vagrant@repoverse:/tmp/dvinstall$ # mg. Backup of solrconfig.xml
vagrant@repoverse:/tmp/dvinstall$ sudo cp \
> /usr/local/solr/solr-8.11.2/server/solr/collection1/conf/solrconfig.xml \
> /usr/local/solr/solr-8.11.2/server/solr/collection1/conf/solrconfig.xml_orig
vagrant@repoverse:/tmp/dvinstall$
vagrant@repoverse:/tmp/dvinstall$ sudo cp \
> schema*.xml \
> /usr/local/solr/solr-8.11.2/server/solr/collection1/conf
vagrant@repoverse:/tmp/dvinstall$ sudo cp \
> solrconfig.xml \
> /usr/local/solr/solr-8.11.2/server/solr/collection1/conf
vagrant@repoverse:/tmp/dvinstall$
```

Next we change the file `jetty.xml` to increase the default value of `requestHeaderSize`

```
vagrant@repoverse:~$ sudo cp /usr/local/solr/solr-8.11.2/server/etc/jetty.xml \
> /usr/local/solr/solr-8.11.2/server/etc/jetty.xml_orig
vagrant@repoverse:~$
vagrant@repoverse:~$ sudo vim /usr/local/solr/solr-8.11.2/server/etc/jetty.xml
vagrant@repoverse:~$
vagrant@repoverse:~$ diff /usr/local/solr/solr-8.11.2/server/etc/jetty.xml \
> /usr/local/solr/solr-8.11.2/server/etc/jetty.xml_orig
71c71
<     <Set name="requestHeaderSize"><Property name="solr.jetty.request.header.size" default="102400" /></Set>
---
>     <Set name="requestHeaderSize"><Property name="solr.jetty.request.header.size" default="8192" /></Set>
vagrant@repoverse:~$
```

Giving Solr more resources to run Solr in production

```
vagrant@repoverse:~$ sudo cp /etc/security/limits.conf /etc/security/limits.conf_orig
vagrant@repoverse:~$
vagrant@repoverse:~$ sudo vim /etc/security/limits.conf
vagrant@repoverse:~$
vagrant@repoverse:~$ diff /etc/security/limits.conf /etc/security/limits.conf_orig
56,60d55
< # Solr (Dataverse)
< solr          soft    nproc           65000
< solr          hard    nproc           65000
< solr          soft    nofile          65000
< solr          hard    nofile          65000
vagrant@repoverse:~$
```

There is extra configuration on the startup (init) script (see below)

```
vagrant@repoverse:~$ grep 'Limit' solr.service
LimitNOFILE=65000
LimitNPROC=65000
vagrant@repoverse:~$
```

Telling Solr to create the "collection1" on startup.

```
vagrant@repoverse:/usr/local/solr/solr-8.11.2/server/solr$ sudo -i -u solr
sudo: unable to change directory to /home/solr: No such file or directory
$ echo "name=collection1" > /usr/local/solr/solr-8.11.2/server/solr/collection1/core.properties
$ exit
vagrant@repoverse:/usr/local/solr/solr-8.11.2/server/solr$
```

### Solr Init Script

Download the Solr startup script

```
vagrant@repoverse:~$ wget https://guides.dataverse.org/en/latest/_downloads/0736976a136678bbc024ce423b223d3a/solr.service
```

Update the script for the correct version of Dataverse

```
vagrant@repoverse:~$ cp solr.service solr.service_orig
vagrant@repoverse:~$
vagrant@repoverse:~$ sed -i 's/11.1/11.2/' solr.service
vagrant@repoverse:~$ diff solr.service solr.service_orig
8,10c8,10
< WorkingDirectory = /usr/local/solr/solr-8.11.2
< ExecStart = /usr/local/solr/solr-8.11.2/bin/solr start -m 1g -j "jetty.host=127.0.0.1"
< ExecStop = /usr/local/solr/solr-8.11.2/bin/solr stop
---
> WorkingDirectory = /usr/local/solr/solr-8.11.1
> ExecStart = /usr/local/solr/solr-8.11.1/bin/solr start -m 1g -j "jetty.host=127.0.0.1"
> ExecStop = /usr/local/solr/solr-8.11.1/bin/solr stop
vagrant@repoverse:~$
```

Enabling the `systemd` scripts

```
vagrant@repoverse:~$ sudo cp solr.service /etc/systemd/system
vagrant@repoverse:~$ sudo systemctl daemon-reload
vagrant@repoverse:~$ sudo systemctl enable solr
Created symlink /etc/systemd/system/multi-user.target.wants/solr.service → /etc/systemd/system/solr.service.
vagrant@repoverse:~$
```

A quick test (starting and stopping) show that Solr seems to be working

```
vagrant@repoverse:~$ sudo systemctl start solr.service
vagrant@repoverse:~$
vagrant@repoverse:~$ sudo systemctl status solr.service
● solr.service - Apache Solr
     Loaded: loaded (/etc/systemd/system/solr.service; disabled; vendor preset: enabled)
     Active: active (running) since Sat 2023-02-25 11:16:18 UTC; 17s ago
    Process: 1262 ExecStart=/usr/local/solr/solr-8.11.2/bin/solr start -m 1g -j jetty.host=127.0.0.1 (code=exited, status=0/SUCCESS)
   Main PID: 1314 (java)
      Tasks: 42 (limit: 9505)
     Memory: 1.1G
        CPU: 12.033s
     CGroup: /system.slice/solr.service
             └─1314 java -server -Xms1g -Xmx1g -XX:+UseG1GC -XX:+PerfDisableSharedMem -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=250 -XX:+UseLargePages -XX:+>

Feb 25 11:16:14 repoverse systemd[1]: Starting Apache Solr...
Feb 25 11:16:14 repoverse solr[1262]: Warning: Available entropy is low. As a result, use of the UUIDField, SSL, or any other features that require
Feb 25 11:16:14 repoverse solr[1262]: RNG might not work properly. To check for the amount of available entropy, use 'cat /proc/sys/kernel/random/entropy_avail'.
Feb 25 11:16:18 repoverse solr[1262]: [158B blob data]
Feb 25 11:16:18 repoverse solr[1322]: Started Solr server on port 8983 (pid=1314). Happy searching!
Feb 25 11:16:18 repoverse solr[1262]: [14B blob data]
Feb 25 11:16:18 repoverse systemd[1]: Started Apache Solr.
vagrant@repoverse:~$
vagrant@repoverse:~$ sudo systemctl stop solr.service
vagrant@repoverse:~$
vagrant@repoverse:~$ sudo systemctl status solr.service
● solr.service - Apache Solr
     Loaded: loaded (/etc/systemd/system/solr.service; disabled; vendor preset: enabled)
     Active: inactive (dead)

Feb 25 11:16:18 repoverse solr[1262]: [158B blob data]
Feb 25 11:16:18 repoverse solr[1322]: Started Solr server on port 8983 (pid=1314). Happy searching!
Feb 25 11:16:18 repoverse solr[1262]: [14B blob data]
Feb 25 11:16:18 repoverse systemd[1]: Started Apache Solr.
Feb 25 11:16:46 repoverse systemd[1]: Stopping Apache Solr...
Feb 25 11:16:47 repoverse solr[1405]: Sending stop command to Solr running on port 8983 ... waiting up to 180 seconds to allow Jetty process 1314 to stop gracefull>
Feb 25 11:16:49 repoverse solr[1405]: [56B blob data]
Feb 25 11:16:49 repoverse systemd[1]: solr.service: Succeeded.
Feb 25 11:16:49 repoverse systemd[1]: Stopped Apache Solr.
Feb 25 11:16:49 repoverse systemd[1]: solr.service: Consumed 12.986s CPU time.
vagrant@repoverse:~$
```

### Securing Solr

TODO: Not configuring a firewall at this poing.

## jq

Installed `jq`

```
vagrant@repoverse:~$ sudo apt install jq
(...)
Do you want to continue? [Y/n]
Get:1 https://deb.debian.org/debian bullseye/main amd64 libonig5 amd64 6.9.6-1.1 [185 kB]
Get:2 https://deb.debian.org/debian bullseye/main amd64 libjq1 amd64 1.6-2.1 [135 kB]
Get:3 https://deb.debian.org/debian bullseye/main amd64 jq amd64 1.6-2.1 [64.9 kB]
Fetched 384 kB in 1s (446 kB/s)
(...)
```

## ImageMagick

Installing ImageMagick

```
vagrant@repoverse:~$ sudo apt install imagemagick
```

Double checking the location of `convert` utility is in the right place, and here we are good:

```
vagrant@repoverse:~$ ls /usr/bin/convert*
/usr/bin/convert  /usr/bin/convert-im6  /usr/bin/convert-im6.q16
vagrant@repoverse:~$
```

## R

Installing R

```
vagrant@repoverse:~$ sudo apt install r-base r-base
```

### R Packages

Installing R packages

- R2HTML
- rjson
- DescTools
- Rserve
- haven

#### Dependencies

Installing the following [packages as dependency](https://stackoverflow.com/questions/65958533/r-error-dependencies-xml2-httr-are-not-available-for-package-linux-mint-2) for `DescTools` package

```
sudo apt install build-essential libcurl4-gnutls-dev libxml2-dev libssl-dev
```

#### Installing R Packages

Locating the R libraries

```
vagrant@repoverse:~$ ls -l /usr/lib/R/library
total 120
drwxr-xr-x  8 root root 4096 Feb 25 13:38 KernSmooth
drwxr-xr-x 10 root root 4096 Feb 25 13:38 MASS
(...)
```

Installing the libraries are _root_

```
vagrant@repoverse:~$ sudo R

R version 4.0.4 (2021-02-15) -- "Lost Library Book"
(...)
> install.packages("R2HTML", repos="https://cloud.r-project.org/", lib="/usr/lib/R/library")
trying URL 'https://cloud.r-project.org/src/contrib/R2HTML_2.3.3.tar.gz'
Content type 'application/x-gzip' length 315809 bytes (308 KB)
(...)
> install.packages("rjson", repos="https://cloud.r-project.org/", lib="/usr/lib/R/library" )
(...)
> install.packages("DescTools", repos="https://cloud.r-project.org/", lib="/usr/lib/R/library")
(...)
> install.packages("Rserve", repos="https://cloud.r-project.org/", lib="/usr/lib/R/library")
(...)
> install.packages("haven", repos="https://cloud.r-project.org/", lib="/usr/lib/R/library")
(...)
```

## Rserve

The Dataverse Software uses Rserve to communicate to R.

Cloned the `dataverse` repository

```
vagrant@repoverse:~$ sudo apt install git
vagrant@repoverse:~$ git clone -b master https://github.com/IQSS/dataverse.git
```

Edited the `rserve-setup.sh` to replace the old `init.d` to use `systemd` instead

```
vagrant@repoverse:~/dataverse/scripts/r/rserve$ cp rserve-setup.sh rserve-setup.sh_orig
vagrant@repoverse:~/dataverse/scripts/r/rserve$ diff rserve-setup.sh rserve-setup.sh_orig
42c42
< if [ ! -f /usr/lib/systemd/system/rserve.service ]
---
> if [ ! -f /etc/init.d/rserve ]
44c44,46
<     echo "Installing Rserve startup file (systemd)."
---
>     echo "Installing Rserve startup file."
>     install rserve-startup.sh /etc/init.d/rserve
>     chkconfig rserve on
46c48
<     echo "systemctl start rserve"
---
>     echo "  service rserve start"
48,58c50
<     echo "Copying 'rserve.service' to '/usr/lib/systemd/system'"
<     cp rserve.service /usr/lib/systemd/system
<     echo "Running 'systemctl daemon-reload'"
<     systemctl daemon-reload
<     echo "Running 'systemctl enable rserve'"
<     systemctl enable rserve
<     echo "Running 'systemctl start rserve'"
<     systemctl start rserve
<     echo "Sleeping 2 minutes"
<     sleep 120
<     echo "done."
---
>     echo "If this is a RedHat/CentOS 7/8 system, you may want to use the systemctl file rserve.service instead (provided in this directory)"
vagrant@repoverse:~/dataverse/scripts/r/rserve$
```

Finally we run the script

```
vagrant@repoverse:~/dataverse/scripts/r/rserve$ sudo ./rserve-setup.sh

Configuring Rserve.


checking if rserve user already exists:

installing Rserv configuration file.

Installing Rserve password file.
Please change the default password in /etc/Rserv.pwd
(and make sure this password is set correctly as a
JVM option in the glassfish configuration of your DVN)

Installing Rserve startup file (systemd).
You can start Rserve daemon by executing
systemctl start rserve

Copying 'rserve.service' to '/usr/lib/systemd/system'
Running 'systemctl daemon-reload'
Running 'systemctl enable rserve'
Created symlink /etc/systemd/system/multi-user.target.wants/rserve.service → /lib/systemd/system/rserve.service.
Running 'systemctl start rserve'
Sleeping 2 minutes
done.

Successfully installed Dataverse Rserve framework.
vagrant@repoverse:~/dataverse/scripts/r/rserve$
```

Check if the service is running

```
vagrant@repoverse:~/dataverse/scripts/r/rserve$ systemctl status rserve
● rserve.service - Rserve
     Loaded: loaded (/lib/systemd/system/rserve.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2023-02-26 18:44:45 UTC; 2min 9s ago
    Process: 26228 ExecStartPre=/usr/bin/mkdir -p /var/run/rserve (code=exited, status=0/SUCCESS)
    Process: 26229 ExecStartPre=/usr/bin/chown -R rserve:rserve /var/run/rserve (code=exited, status=0/SUCCESS)
    Process: 26230 ExecStart=/usr/bin/R CMD Rserve --quiet --vanilla --RS-conf /etc/Rserv.conf --RS-pidfile /var/run/rserve/rserve.pid (code=exited, stat>
   Main PID: 26238 (Rserve)
      Tasks: 1 (limit: 9505)
     Memory: 37.1M
        CPU: 263ms
     CGroup: /system.slice/rserve.service
             └─26238 /usr/lib/R/bin/Rserve --quiet --vanilla --RS-conf /etc/Rserv.conf --RS-pidfile /var/run/rserve/rserve.pid
vagrant@repoverse:~/dataverse/scripts/r/rserve$
```

## Counter Processor

Counter Processor is required to enable Make Data Count metrics in a Dataverse installation.

### Installing

```
wget https://github.com/CDLUC3/counter-processor/archive/v0.1.04.tar.gz
tar tzf v0.1.04.tar.gz
sudo tar xzf v0.1.04.tar.gz -C /usr/local/
```

### Creating Counter User

```
vagrant@repoverse:~$ sudo useradd counter
vagrant@repoverse:~$ sudo chown -R counter:counter /usr/local/counter-processor-0.1.04/
```

Installing `python-venv` as dependency

```
vagrant@repoverse:/usr/local/counter-processor-0.1.04$ sudo apt-get install python3-venv
vagrant@repoverse:/usr/local/counter-processor-0.1.04$ sudo -i -u counter
$ cd /usr/local/counter-processor-0.1.04
$ python3 -m venv venv
$ . venv/bin/activate
(venv) $ pip install -r requirements.txt
```

## Installation

Installing as dependency

```
vagrant@repoverse:~$ sudo apt install python3-psycopg2
vagrant@repoverse:~$ sudo apt install postgresql-server-dev-all
```
