---
title: "Wslbackup"
date: 2017-07-11T13:38:22-04:00
draft: false
author: Brian Algeri
---


# Windows Subsystem for Linux Backup

Here is one solution mostly staying in the Windows Subsystem for Linux\
(WSL). Shown from the command line for easier understanding. In practice\
I would automate this from a script and place a datecode in the filenames.
<!--more-->

Replace dyax with your user.

## To Backup:

### In the WSL:

Su to root:
```text
su root
cd /home
```

Backup the system:
```text
find / -maxdepth 1 -type d \( -path /dev -o -path /sys -o -path /proc\
-o -path /home -o -path /mnt -o -path /tmp \) -prune -o ! -wholename /\
-exec sh -c 'find "$1" -depth' {} {} \; | cpio -ovac > sysbackup.cpio
```

Backup the user:
```text
find dyax -depth | cpio -ovac > dyaxbackup.cpio
```

Compress the backups:
```text
gzip -9 systembackup.cpio &
gzip -9 dyaxbackup.cpio &
```

When the jobs finish store the backup files:
```text
cp systembackup.cpio.gz /mnt/d/backups
cp dyax-backup.cpio.gz /mnt/d/backups
rm systembackup* dyaxbackup*
```

Create a directory list to help automate the restore:
```text
find / -maxdepth 1 -type d \( -path /dev -o -path /sys -o -path /proc\
-o -path /home -o -path /mnt -o -path /tmp \) -prune -o -print > dirlist
sed -i s#/##g dirlist
sed -i /init/d dirlist
mv dirlist ~
```

Exit from root:
```text
exit
```

## To Restore:

### In the WSL:

Su to root:
```text
su root
```

Extract the system backup files:
```text
cd /home
mkdir restore
cd restore
cp /mnt/d/backups/systembackup.cpio.gz .
cp ~/dirlist .
zcat systembackup.cpio.gz | cpio -ivdm --no-absolute-filenames
rm systembackup.cpio.gz
rm init
cd ..
```

Extract and restore the user files:
```text
cp /mnt/d/dyaxbackup.cpio.gz .
rm -rf dyax
zcat dyaxbackup.cpio.gz | cpio -ivdm
```

Exit from root:
```text
exit
```
Exit the system:
```text
$ exit
```

# On Windows 10:

## Resore the system files:

### From the Command Prompt:
```text
cd C:\Users\dyax\AppData\Local\lxss\rootfs
for /f %i in (..\home\restore\dirlist) do rmdir /s /q %i
cd ..\home\restore
for /f %i in (dirlist) do move /y %i ..\..\rootfs
del dirlist
cd ..
rmdir restore
```

Exit the Command Prompt:
```text
exit
```

### Finished:
Note:  You must move the system files, copy will not work -- file links


