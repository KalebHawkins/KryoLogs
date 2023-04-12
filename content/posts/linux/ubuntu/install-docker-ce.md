---
title: Install Docker CE (Ubuntu 20.04)
date: 2023-02-10
author: Kaleb Hawkins
description: Install Docker CE on Ubuntu 20.04
tags:
  - ubuntu
  - docker
  - containers
---

- [Installation Steps](#installation-steps)
- [Addition Steps](#addition-steps)

## Installation Steps 

Remove older versions of Docker if installed.

```bash
sudo apt remove docker docker-engine docker.io containerd runc
```

Install the Docker repository.

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

Install Docker CE.

```bash
sudo apt update -y
sudo apt install docker-ce docker-ce-cli containerd.io -y
```

Check the service status.

```bash
sudo systemctl status docker 
```

Test Installation.

```bash
sudo docker run hello-world
```

## Addition Steps

Add user to docker group.

```bash
sudo usermod -aG docker $USER
```

> Log out and log back in so that your group membership is re-evaluated.
> If testing on a virtual machine, it may be necessary to restart the virtual machine for changes to take effect.
> On a desktop Linux environment such as X Windows, log out of your session completely and then log back in.