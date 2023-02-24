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
