---
weight: 999
title: "Vmware Template"
description: ""
icon: "article"
date: "2024-05-30T14:39:33-05:00"
lastmod: "2024-05-30T14:39:33-05:00"
draft: false
toc: true
---

## Prerequisites

* Access to Red Hat's access portal to download the Red Hat ISO.
* An active Red Hat Enterprise Linux subscription.

## Create the Virtual Machine

Sign into vCenter.  

Right-click the cluster you want to create the virtual machine on.  

Select `New Virtual Machine`.

Set the name and select a folder to place the template in.

Select the compute resource to place the template on.

Select the datastore to place the template on.

Select Compatibility.

Select the guest OS.  
* Guest OS Family: Linux  
* Guest OS Version: Red Hat Enterprise Linux 8 (64-bit)

Customize the hardware. 

> Use a 100Gb disk for the system OS disk. This will be what the partition scheme later on is based off of.

> Make sure the SCSI Controller is set to VMware Paravirtual.  


Create the virtual machine.

## Prepare for Red Hat Installation

Download the REd Hat 8 ISO from Red Hat's access portal.

Upload the ISO to a datastore accessible to the host you put the template virtual machine on.

Right-click on the template virtual machine.

Click `Edit Settings` then connect the `CD/DVD drive`.

Drop down the `CD/DVD drive` section

Select the Datastore ISO file on the first option.

Use the `BROWSE...` button to select the ISO you uploaded. 

Start the virtual machine.

## Red Hat Installation

For the Installation screen navigate through the prompts selecting the correct settings.

Under `Date & Time` select your timezone. (NTP will be configured later)

Under `Software Selection` select minimal install.

Under `Network & Host Name` go ahead and connect your network adapter by clicking `Configure...`. 

> Set the device name to `ens192`.  
> Set the `IPv4 Settings` to the desired values. If this is going to be statically set then the `Method` should be set to `Manual`.  
> Set the `IPv6 Settings` to `Disabled`.  

Under `Installation Destination` in the `Storage Configuration` section select `Custom` the click `Done`. 

In the menu presented click the `+` button to add a mount point. See below for what the layout should look like.

| Mount Point | Size | File System | Device Type | 
|-------------|------|-------------|-------------|
| /boot       | 1G   | xfs         | Standard Partition |
| /home       | 20G  | xfs         | LVM         |
| /var        | 25G  | xfs         | LVM         |
| /           | 55G  | xfs         | LVM         |
| swap        | 4G   | xfs         | Swap        |

Click `Done` once all is configured. Create a user if desired and set the root password from the menu options.

## Template Configuration

Use the below to perform the template configuration.

```bash
# Register to Red Hat.
subscription-manager register --username <username> --password <password> --autosubscribe
# Update all system packages.
dnf update -y 
# Get some essential tools.
dnf install dnf-utils perl cloud-init python3-libselinux git bash-completion -y
# Disable `cloud-init`.
touch /etc/cloud/cloud-init.disabled
# Seal the template.
curl -L -o /opt/seal https://gist.githubusercontent.com/KalebHawkins/9a0877a6ba7bb0405ca32dc26b9a0004/raw/2455b03c49cab8c065335b14895905628d9313e9/Red%2520Hat%2520Template%2520Seal%2520Scripts
chmod 700 /opt/seal
/opt/seal
```