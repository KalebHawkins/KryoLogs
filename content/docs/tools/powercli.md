---
weight: 999
title: "PowerCLI"
description: ""
icon: "article"
date: "2024-05-30T14:56:44-05:00"
lastmod: "2024-05-30T14:56:44-05:00"
draft: false
toc: true
---

## Overview

Vmware PowerCLI is a Powershell module used interact with vCenter amd vSphere APIs. This allows for simple automation of mundane, redundant tasks, like inventorying servers. 

## Installation

To install PowerCLI open up a powershell window and run the following command.

```powershell
Install-Module VMware.PowerCLI -Scope CurrentUser -AllowClobber
```

## Using PowerCLI

Getting connected to your cluster can be performed with one command.


```powershell
Connect-ViServer <vcenter.domain.com> -Credentials $(Get-Credentials) -Force
```

Get a list of your clusters, hosts, and vms.

```powershell
Get-Cluster
Get-Host
Get-VM
```

There are many additional commands that can be used to automate entire infratructure deployments.