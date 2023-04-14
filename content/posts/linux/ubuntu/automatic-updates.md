---
title: Ubuntu 20.04 - Enable Automatic Updates 
date: 2023-02-10
author: Kaleb Hawkins
description: Setup automatic updates for Ubuntu 20.04
tags:
  - ubuntu
---

Update and install required packages.

```bash
sudo apt update && sudo apt upgrade
sudo apt install unattended-upgrades
```

Turn on unattended updates.

```bash
sudo dpkg-reconfigure -plow unattended-upgrades
```