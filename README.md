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


