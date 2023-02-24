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

