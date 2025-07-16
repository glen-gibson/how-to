# Fedora CoreOS Install

## Table of Contents
- [Install Media](#install-media)
- [Creating Ignition Files](#creating-ignition-files)
  - [Example file](#example-file)
- [Serving Ignition Files](#serving-ignition-files)
- [CoreOS Installer](#coreos-installer)

## Install Media
The first thing we need to install Fedora CoreOS is some actual install media!  

My personal preference is to use an ISO, that way I know it'll generally work, whether it be a Virtual Machine or a Bare Metal install.

Later on if I feel confident, I may then just use `qcow` or raw disk images, especially made for Hypervisors.

You can get the Live DVD install media for Fedora CoreOS here:
https://fedoraproject.org/coreos/download

## Creating Ignition Files
After we've downloaded and booted from the Fedora CoreOS Live DVD we need to run the installer, which requires an ignition file.

The easiest way to create an Ignition file is to write a Butane YAML file first, they generally have a `.bu` file extension.  

The reason I say this is that Ignition files are typical JSON format and are much easier to make mistakes in when trying to write raw JSON.  Ignition files have a `.ign` extension.

### Example file
The following provides a working example of a Butane file that performs custom partitioning of the first disk found on an EFI system.  It also sets an initial SSH key so that an administrator can log in as the default `core` user with their corresponding SSH private key.
```
variant: fcos
version: 1.6.0
storage:
  disks:
  - device: /dev/disk/by-id/coreos-boot-disk
    wipe_table: true
    partitions:
    - number: 2
      label: EFI-SYSTEM
      size_mib: 550
      type_guid: C12A7328-F81F-11D2-BA4B-00A0C93EC93B
    - number: 3
      label: boot
      start_mib: 0
      size_mib: 1024
    - number: 4
      label: root
      start_mib: 0
      size_mib: 0
  filesystems:
    - device: /dev/disk/by-partlabel/EFI-SYSTEM
      format: vfat
      label: EFI-SYSTEM
    - device: /dev/disk/by-partlabel/boot
      wipe_filesystem: true
      format: ext4
      label: boot
    - device: /dev/disk/by-partlabel/root
      wipe_filesystem: true
      format: ext4
      label: root
  files:
    - path: /etc/hostname
      mode: 0644
      contents:
        inline: fcos-test01
passwd:
  users:
    - name: core
      ssh_authorized_keys:
        - ssh-ed25519 AAAAC.... admin@WORKSTATION # your key goes here
```
Save the file as [`fcos_server.bu`](fcos_server.bu) (this can really be anything).  After that, we'll transpile it using the latest Butane container image via `podman`.  We'll name the output file `fcos_server.ign` to be consistent.  
Here is the command:
```
podman run --interactive --rm \
        --security-opt label=disable \
        --volume "${PWD}:/pwd" \
        --workdir /pwd quay.io/coreos/butane:release \
        --pretty \
        --strict fcos_server.bu > fcos_server.ign
```
## Serving Ignition Files
The easiest way to serve up your example file is to just serve it up from Python using the `http` module.

To do this, create a directory, e.g. `ignition` and copy the ignition file you created earlier into it.
```
mkdir ./ignition
cp ./fcos_server.ign ./ignition
```
Change directory into the `ignition` directory you created and then run up the Python server using the http module. The following will start an HTTP server listening on port 8000
```
cd ./ignition
python -m http.server 8000
```
__IMPORTANT:  Serving up files over a plain unencrypted HTTP server is a security risk and should not be used in a production environment.__

## CoreOS Installer
Our server is booted from the Live DVD, our ignition file is created and our webserver is listening on port 8000.  The final step is to run the CoreOS installer.

```
sudo coreos-installer install /dev/sda \
    --ignition-url http://ip-of-ignition-server:8000/fcos_server.ign \
    --insecure-ignition
```
The system will let you know when it completes, you can reboot and remove the install media and begin configuring your newly provisioned Fedore CoreOS Server.

## References
https://docs.fedoraproject.org/en-US/fedora-coreos/
https://docs.fedoraproject.org/en-US/fedora-coreos/bare-metal/
https://docs.fedoraproject.org/en-US/fedora-coreos/producing-ign/
https://coreos.github.io/butane/
https://coreos.github.io/butane/config-fcos-v1_6/