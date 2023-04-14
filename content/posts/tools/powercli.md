---
title: VMWare & Powershell - PowerCLI
date: 2023-02-10
author: Kaleb Hawkins
description: Setup guide for VMWare's PowerCLI
tags:
  - windows
  - powershell
  - vmware
  - powercli
---

- [Overview](#overview)
- [Installation](#installation)
- [Using PowerCLI](#using-powercli)

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