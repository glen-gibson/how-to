# Cayo Proxmox Template with Cloud-Init!

### In this Guide:
- [Introduction](#introduction)
- [My Template Creation Steps](#my-template-creation-steps)
  - [My Template Machine](#my-template-machine)
- [Deployment with Cloud-Init](#deployment-with-cloud-init)
- [My Cloud-Init Settings](#my-cloud-init-settings)  
</br>

## Introduction
This is going to be a little rough, just so I get it out there, then I can tidy it up a bit...

Most homelab-ers will know about [Proxmox VE](https://www.proxmox.com/en/products/proxmox-virtual-environment/overview) and how awesome of a distro it is for running up and managing VM's & LXC containers.  This guide will assume a decent level of understanding in how to use Proxmox.

### My Template Machine
I tend to keep most things standard except for using `OVMF (UEFI)` instead of `SeaBIOS` and `q35` instead of `i440fx` for the Virtual Machine I'm building.

## My Template Creation Steps  

1.  Standard `podman` based install - __do not reboot__.
2.  Do the following (assuming `/dev/sda3` is sysroot for the new VM):
```bash
sudo mkdir /mnt/cayo
sudo mount /dev/sda3 /mnt/cayo
cd /mnt/cayo/ostree/boot.1.1/default/[big_long_string]/0/etc/pki/akmods/certs/
sudo mokutil --import akmods-ublue.der
[set the password for importing on reboot]
reboot
```
3.  Go through the blue screen MOK wizard - __don't reboot just yet__.
4.  When you hit reboot at the end of the wizard, hard power off the VM.
5.  Reduce the RAM as it's probably 12GB, I set mine to 4GB
6.  Convert VM to template in Proxmox.

## Deployment with Cloud-Init  

1.  Right click the template and choose "Clone"
2.  Fill out the appropriate boxes and just to be sure, I chose "Full Clone" as the mode to make it completely independent of the template.
3.  Edit the new cloned VM's hardware and add a `CloudInit Drive` 
4.  Go to the new cloned VM's `Cloud-Init` section and fill out the appropriate boxes.
5.  Boot the new cloned VM.  
</br>

## My Cloud-Init Settings

| Key               | Value                                     |
| ----------------- | ----------------------------------------- |
| User              | cayo                                      |
| Password          | none                                      |
| DNS domain        | yourdomain.com                            |
| DNS servers       | ip.of.your.dns                            |
| SSH public key    | [pub key from .ssh]                       |
| Upgrade packages  | No                                        |
| IP Config (net0)  | ip=ip.of.the.machine/CIDR, gw=ip.of.your.gw    |