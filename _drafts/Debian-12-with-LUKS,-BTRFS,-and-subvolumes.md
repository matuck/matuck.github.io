---
layout: post
section-type: post
title: Debian 12 with LUKS, BTRFS, and subvolumes
category: Tech
tags: [ 'linux', 'debian', 'security', 'luks' ]
---
Debian is one of my favorite distributions.  The installer leaves a little to be desired with setting up BTRFS and LUKS.  This tutorial is designed to walk you through the steps of setting up LUKS and BTRFS with subvolumes.   

## Prepartioning
For this part you will need Debian installation medium.  The process is pretty simple but you will use the expert Install instread of the normal install.

* Boot with your install media
* At prompt choose Advanced options
* Choose Expert install

This will launch the text based Debian installer.
* Go through and choose your language
* Configure keyboard
* Load installer components
  *  If you do not select anything here it loads the default modules.
* Detect your network
* Configure on network
* Configure the clock

## Installer Partioning
In this section we will setup the basic encrypted volume and setup the base BTRFS file system.

* Detect the disks
* Partion the disks
  * Manual partioning
    * Configure 1st partition as efi partions 512mb
    * Configure 2nd partition as boot ext2 512mb
    * Configure swap partition if wanted
    * Configure an encrypted volume
      * Create encrypted volume
        * Choose your remaining freespace
          * You can cancel the wipe if you want(in fact its recommended if this is an ssd)
        * Enter in your encryption password
    * Select the encrypted partion
    * Change it to use as: btrfs
    * Set mount point to /
  * Finish partioning and write changes to disk

## Outside Installer Partioning
This is the most complicated part of the install.  Do not continue the noromal install process until you complete this part.  These are manual steps that have to be completed from outside the installer.  In my case the drive I used is nvme0n1p.  This may be different on your system so please repalce that.
* Get to a terminal by doing ctrl+alt+f3
* Hit enter to activate the console
* Unmount the partions
  ```bash
  umount /target/boot/efi
  umount /target/boot
  umount /target
  ```
* Temporarialy remount
  ```bash
  mount /dev/mapper/nvme0n1p3_crypt /mnt
  ```
* Create your subvolumes  
  ```bash
  cd /mnt
  btrfs subvolume create @
  btrfs subvolume create @home
  btrfs subvolume create @var
  btrfs subvolume create @snapshots
  ```
* Mount your new filesystems back to target
  ```bash
  mount -o noatime,compress=lzo,space_cache,subvol=@ /dev/mapper/nvme0n1p3_crypt /target
  mkdir -p /target/{home,var,snapshots}
  mount -o noatime,compress=lzo,space_cache,subvol=@home /dev/mapper/nvme0n1p3_crypt /target/home
  mount -o noatime,compress=lzo,space_cache,subvol=@snapshots /dev/mapper/nvme0n1p3_crypt /target/snapshots
  mount -o noatime,compress=lzo,space_cache,subvol=@var /dev/mapper/nvme0n1p3_crypt /target/var
  mkdir /target/boot
  mount /dev/nvme0n1p2 /target/boot
  mount /dev/nvme0n1p1 /target/boot/efi 
  ```
* Copy over old files
  ```bash
  mkdir /target/etc
  cp /mnt/@rootfs/etc/fstab /target/etc
  cp /mnt/@rootfs/etc/crypttab /target/etc/crypttab
  ```
* Update fstab
  ```bash
  nano /target/mount/etc/fstab
  ```

* Cleanup old rootfs
  ```bash
  rm -rf /mnt/@rootfs/etc /mnt/@rootfs/media /mnt/@rootfs/boot
  umount /mnt
  ```

* Hit ctrl+d to logout of terminal
* Hit ctrl+alt+f1 to go back to installer

## Finishing up
  Continue on as a normal debian install

  see next part of this series for adding luks with fido2 on debian
