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

* Create a new virtual machine in proxmox.
  * Give it an unused vmid
  * Set it to a node in your proxmox cluster
  * Give it a minimal template name like fc37-template
  * On OS select do not use any media
  * On system tab check the qemu agent
  * On disk tab remove the existing disk
  * On cpu tab you can set to what you want for your default vm size
  * On the memory tab set to your default size as well
  * On network tab set to your default network settings.
  * Click finish

## Proxmox command line
Now we need to go to the command line.  There are some things that need to be done on the server.  
* SSH to the proxmox node that the vm was created on.
* Get the cloud init image.
  * In  browser on your computer go to [Fedora](https://cloud.fedoraproject.org/) and copy the url for the qcow2 image
  * On your ssh session run wget with the previosyly copied url
	```
	wget https://mirror.arizona.edu/fedora/linux/releases/37/Cloud/x86_64/images/Fedora-Cloud-Base-37-1.7.x86_64.qcow2
	```
  * Resize the disk to your desired size the command qemu-img resize \<image-filename> \<size>
	```
	qemu-img resize Fedora-Cloud-Base-37-1.7.x86_64.qcow2 60G
	```
  * Now import the disk to your template virtual machine with command qm importdisk <vmid> <image-filename> <storage-name>
	```
	qm importdisk 1000 Fedora-Cloud-Base-37-1.7.x86_64.qcow2 local-lvm
	```
* create a vga console so that we have a display when the server is powered on qm set <vmid> --serial0 socket --vga serial0
  ```
	qm set 1000 --serial0 socket --vga serial0
	```

## Edit the Virtual machine settings
Now we need to edit some settings on the virtual machine to finish the template.
* Start with selecting the virtual machine
* Go to the hardware tab
  * Click add
    * select cloud-init drive
    * select an appropriate storage
  * Select the previously imported disk
    * Click edit
    * Click add 
* Go to the cloud init tab
  * set default values for the vm.
* click regenerate image after updating all cloud init settings
* Go to the Options tab
  * Change the boot order to how you would like it set including adding the hard drive and setting higher in the order
  * You may want to change the start at boot option

## Convert to teplate
Right click on the vm and select convert to template after you have confirmed your settings.

## Turn your template into a vm.
* Select your template and right click on it
* Select clone
* Fill out the form with name, vmid, and mode (I prefer full clone over linked linked clone)
* After the clone is complete start vm but do not login at first prompt. Wait for cloud-init to complete
* You will want to install qemu-guest-agent after install reboot the server


## Customize a new template
Login to vm install any packages and put any customizations then run the below commands
```
sudo cloud-init clean
sudo rm -rf /var/lib/cloud/instances
sudo truncate -s 0 /etc/machine-id
sudo poweroff
```
now right click vm and convert to template

## Sources
The sources below are what helped me to lay this out.
https://www.youtube.com/watch?v=MJgIm03Jxdo
https://www.learnlinux.tv/how-to-build-an-awesome-kubernetes-cluster-using-proxmox-virtual-environment/

