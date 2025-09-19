# Fedora CoreOS Proxmox Template!

### In this Guide:
- [Introduction](#introduction)
- [Proxmox Prep-work](#proxmox-prep-work)
- [The Template Machine](#the-template-machine)
  - [Hardware](#hardware)
  - [The Ignition File](#the-ignition-file)
  - [Import a Hard Disk via the CLI](#import-a-hard-disk-via-the-cli)
  - [The Cloud-Init Drive](#the-cloud-init-drive)
  - [Cloud-Init Configuration](#cloud-init-configuration)
- [References](#references)
</br>

## Introduction  
Most homelab-ers will know about [Proxmox VE](https://www.proxmox.com/en/products/proxmox-virtual-environment/overview) an excellent Debian based distribution that makes it very easy create and manage virtual machines & LXC containers.  

_This guide will assume a decent level of understanding of how to use Proxmox as this is not a Proxmox how-to guide._  

## Doesn't Fedora CoreOS use Ignition instead of Cloud-Init?
It sure does, but read on if you want to find out how to kind of trick Proxmox into pushing through settings via both Ignition and Cloud-Init based information.

## Proxmox Prep-work
SSH into the Proxmox VE host server and create a directory to keep the Fedora CoreOS files in.
```bash
# Create directories for our items
mkdir -p /var/coreos/images
mkdir -p /var/coreos/snippets

# Add the parent directory as Proxmox storage so that it's aware of it
pvesm add dir coreos --path /var/coreos --content images,snippets
```

Next we need to download a Fedora CoreOS disk image for Proxmox, fortunately they supply on specifically designed for Proxmox over at [https://fedoraproject.org/coreos/download/].

Here's an example of downloading and decompressing the current stable release at the time of writing:
```bash
# Change directory to where we'll keep our downloaded images
cd /var/coreos/images

# Retrieve the image
wget https://builds.coreos.fedoraproject.org/prod/streams/stable/builds/42.20250901.3.0/x86_64/fedora-coreos-42.20250901.3.0-proxmoxve.x86_64.qcow2.xz

# Decompress the image
unxz fedora-coreos-42.20250901.3.0-proxmoxve.x86_64.qcow2.xz
```

## The Template Machine

### Hardware
Personally, I tend accept most defaults except for using `OVMF (UEFI)` instead of `SeaBIOS` and `q35` instead of `i440fx` for the virtual machines I build.  Run through the remainder of the wizard, but __delete the default hard disk and don't add a new one__.  You'll see why soon; Make a note of the VMID - the first default number is usually 100.  For templates, I like to assign the number 900 for the first, 901 for the second, etc.  I use 100 onwards for real virtual machines.  

So, let's use 900 for this example.

### The Ignition File
I'm not going to go into too much detail on Butane and Ignition.  Basically Butane files are easier for humans to read and edit as they're YAML, Ignition files are JSON.  Butane converts the YAML file into an Ignition JSON file that is usable by Fedora CoreOS.

A minimal Fedora CoreOS butane file can be found [here](./config.bu).  

Butane documentation for the spec at time of writing can be found here [https://coreos.github.io/butane/config-fcos-v1_6/]

The command to do this is:
```bash
podman run --interactive --rm \
        --security-opt label=disable \
        --volume "${PWD}:/pwd" \
        --workdir /pwd quay.io/coreos/butane:release \
        --pretty \
        --strict config.bu > config.ign
```

### Import a Hard Disk via the CLI
This step cannot be performed via the Proxmox Web UI.

```bash
# copies the downloaded qcow2 disk image file contents to raw storage on local-lvm
qm importdisk 900 /var/coreos/images/fedora-coreos-42.20250901.3.0-proxmoxve.x86_64.qcow2 local-lvm
```

If you click into another node and then back into the `Hardware` node of your new VM (900), you'll see that you now have a hard disk named `Ununsed Disk 0` - double click it, make sure it's set to `scsi0` and click `Add`.  

By default, the disk created from the qcow2 image is only 10GB.  Expand it if you need to, using the following:
```bash
# expands the disk we added at scsi0 by an extra 15GB
qm resize 900 scsi0 +15G
```

### The Cloud-Init Drive
This is where the magic happens!  We'll now create a custom Cloud-Init drive via the CLI.
```bash
# Create a Cloud-Init drive on local-lvm storage and attach to IDE2
qm set 900 --ide2 local-lvm:cloudinit

# Tell it about the ignition file - coreos is the storage we added at the start
# config.ign is the name of our ignition file inside the snippets directory
qm set 900 --cicustom vendor=coreos:snippets/config.ign

# Prevent automatic upgrades of this Cloud-Init drive as it'll overwrite our stuff
qm set 900 --ciupgrade 0
```

If you haven't booted the VM, you can now convert this machine to a template.

## Cloud-Init Configuration
Clone the template to a new machine.

On the left, below the `Hardware` item in the web UI for the VM, there's a `Cloud-Init` item.  Click on this and you'll be able to specify some settings that will actually be passed through at first boot.  

The host name you gave the VM when the clone was created is automatically passed through to the VM as its hostname.

At the time of testing, I have successfully used the following items to apply at first boot:
- DNS servers
- IP Config

What I've tried but hasn't worked:
- DNS domain

Haven't tested yet:
- User (mine is in the ignition file)
- Password
- SSH public key (mine is in the ignition file)

</br>

## References
https://docs.fedoraproject.org/en-US/fedora-coreos/provisioning-proxmoxve/  
https://coreos.github.io/butane/config-fcos-v1_6/