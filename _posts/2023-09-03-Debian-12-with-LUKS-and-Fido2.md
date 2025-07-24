---
layout: post
section-type: post
title: Debian 12 with LUKS and Fido2
category: Tech
tags: [ 'linux', 'debian', 'security', 'luks', 'fido2' ,'tutorial' ]
---
<img class="floatimageleft borderless" src="/img/debianlogo.svg"  width="200px" />LUKS is a great way to keep your data safe when it is at rest.  I was seeking a way to keep my data safe and make it convienent to do so.  If you have a yubikey 5 series or later this can be done with fido2.  This tutorial will show the process I used to set this up.  In my case the drive I am useing for this is nvme0n1p.  You will need to substitue that as needed.

## Backups
I recommend doing some backups.  Before you get started.  You will need these if you need to recover.

### LUKS Header
Save your existing LUKS header to external media incase you need to go back and restore it.
``` bash
cryptsetup luksHeaderBackup /dev/nvme0n1p3 --header-backup-file <external_media_path>/luks_backup.bin
```
### Initrd backups
We will be updating some of your intramfs.  This could make your system unbootable.  If you make copy of the files in the same directory, you can easily boot back in from grub. Inside /boot there will be one or more files that start with initrd.  In my case the files look like initrd.img-6.4.0-3-amd64.  Copy each of your initrd files and append with .bak.
```bash
cd /boot
cp initrd.img-6.4.0-3-amd64 initrd.img-6.4.0-3-amd64.bak
```

##  Ensure pin is setup on your yubikey
If you wish for pin entry to unlock your yubikey, make sure your pin is setup.  If you have already setup pin, you can skip this.
```bash
ykman fido access change-pin
```

## Enroll the Yubikey to the LUKS drive
The next step is to add your yubikey to the luks encrypted partition.
```bash
systemd-cryptenroll /dev/nvmeon1p3 --fido2-device=auto --fido2-with-client-pin=yes
```
This will prompt for the current password for the partition.  Enter that.  Once that is complete make sure your yubikey is plugged in if it isn't already.  It will then ask for your PIN.  Enter that.  It will requests you to touch the yubikwy with the blinking light.

## Modify Crypttab
Your crypttab file will need to be modified to allow the use of the yubikey.  The orignal file looks like
```conf
nvme0n1p3_crypt UUID=6dd1a575-8ae3-4fd3-89fe-46b5dd274541 none luks,discard
```
The modified file looks like.
```conf
nvme0n1p3_crypt UUID=6dd1a575-8ae3-4fd3-89fe-46b5dd274541 none luks,discard,fido2-device=auto
```

## Setup dracut
By default Debian uses initramfs-tools.  These tools use the old cryptsetup and will now allow fido2.  Because of this we will need to setup dracut.  This is not quite as simple as just installing dracut some configuration will need to be done to get everything working.
### Install dracut fido2-tools
This part is simple we use apt to install dracut this will update your initrd files but yoursystem will not boot correctly yet.  Follow the next section.
```bash
apt install dracut fido2-tools
```
### Configure dracut
Create a file called 11-fido.conf in /etc/dracut.conf.d .  Make sure this file user and group are root.  Contents of the file are below 
```conf
## Spaces in the quotes are critical.
install_optional_items+=" /usr/lib/x86_64-linux-gnu/libz.so.* "

## Ugly workround because the line above doesn't fetch
## dependencies of libfido2.so
install_items+=" /usr/bin/fido2-token /etc/crypttab "

# Required detecting the fido2 key
install_items+=" /usr/lib/udev/rules.d/60-fido-id.rules /usr/lib/udev/fido_id "
add_dracutmodules+=" fido2 "
```
### Regenerate dracut
After everything is installed and configured the initrd files have to be regenerated.  This is easy its a single command
```bash
dracut -f
```
## Update Grub
System will still have trouble booting, because it needs help finding the encrypted drive. Grub will need to be updated.
### Reconfigure Grub
In your /etc/default/grub the GRUB_CMDLINE_LINUX.  We need to add an entry for rd.luks.name.  An excert from my file is below
```conf
...
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
GRUB_CMDLINE_LINUX="rd.luks.name=6dd1a575-8ae3-4fd3-89fe-46b5dd274541=nvme0n1p3_crypt" 
# If your computer has multiple operating systems installed, then you
...
```
### Apply the reconfigure
This is a simple oneline command.
```bash
update-grub
```
## Test Everything out
From here everything should work.  Reboot your system.  You will be prompted for your yubikey pin.  Enter your pin and hit enter.  Make sure your yubikey is plugged in.  It should start blinking.  Touch your yubikey, and it should continue the boot.
## Emergency roll back
If you are not prompted for the pin or it doesnt unlock, reboot your machine again.  At the grub prompt hit e.  On the initrd line add .bak to the initrd image created.
