---
layout: post
section-type: post
title: Getting Started With Cloud Init On Proxmox
category: Tech
tags: [ 'VM', 'Proxmox', 'Cloud-Init', 'HomeLab']
---

I have been installing lots of virtual machines lately.  Until now that has been a problematic.  I want to steamline the process of creating new virtual machines.  To solve much of the initial setup I am going to start using virtual machine templates with cloud-init.  This will allow virtual machines to be spun up much quicker with updates automatically ran.  This blog is going to server as an outline and tutorial of this process.  

I use Fedora primarily for my servers so this tutorial will be based on Fedora 37 but should be easily adapted to other distro's with cloud init. 

## First we need to create the virtual machine in Proxmox


Create a new virtual machine in proxmox.
Give it to vmid that is not in use.  
Set it to a node in your proxmox cluster
Give it a minimal template name like fc37-template
on Os do not select any cd image choose do not use any media
on system tab check the qemu agent
on disk tab remove the existing disk
on cpu tab you can set to what you want for your default vm size
on the memory tab set to your default size as well
on network tab set to your default network settings.
click finish

on the hardware tab for the virutal machine
add cloudinit drive.
select an appropriate storage
go to the cloud init tab and set default values for the vm.
click regenerate image after updating all cloud init settings

now ssh into the proxmox server the vm is on
use wget to download your cloud image from [Fedora](https://cloud.fedoraproject.org/) download the .img or qcow2 version
if downloaded img mv to a qcow2 file
now resize the disk to the size you are going to want the vm to be
```
qemu-img resize <image-filename> <size>
qemu-img resize fedoara37.qcow2 60G
```
now import the image into a vm.  with the below command
```
qm importdisk <vmid> <image-filename> <storage-name>
qm importdisk 900 fedora37.qcow2 nfs
```

create a vga console so that we have a display when the server is powered on 
```
qm set <vmid> --serial0 socket --vga serial0
```

now go back to the vm and on the hardware tab
select the disk and click edit then click add

Go to options and change the boot order to how you would like it set including adding the hard drive and setting higher in the order
you may want to change the start at boot option

now right click on the vm and convert to template once you have confirmed all your settings

now to create a vm from the template right click the template and select clone
fillout the form with name and vm id and the mode.  I prefer full clone over linked clone
Once the clone is done you can start your new vm.  Do not login at first login prompt.  Wait for cloud-init to finish
You will want to install qemu-guest-agent after install reboot the server


Customize a new template
login to vm install any packages and put any customizations then run the below commands
```
sudo cloud-init clean
sudo rm -rf /var/lib/cloud/instances
sudo truncate -s 0 /etc/machine-id
sudo rm /var/lib/dbus/machine-id
sudo ln -s /etc/machine-id /var/lib/dbus/machine-id
sudo poweroff
```
now right click vm and convert to template

Sources
https://www.youtube.com/watch?v=MJgIm03Jxdo
https://www.learnlinux.tv/how-to-build-an-awesome-kubernetes-cluster-using-proxmox-virtual-environment/

