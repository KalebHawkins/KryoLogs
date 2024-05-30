---
weight: 999
title: "Minikube"
description: ""
icon: "article"
date: "2024-05-30T14:56:27-05:00"
lastmod: "2024-05-30T14:56:27-05:00"
draft: false
toc: true
---

## Overview

Minikube is local Kubernetes, focusing on making it easy to learn and develop for Kubernetes.
All you need is Docker (or similarly compatible) container or a Virtual Machine environment, and Kubernetes is a single command away: `minikube start`.

## Installation

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

## Uninstallation

```bash
sudo rm /usr/local/bin/minikube
```

## Customize Cluster

Get configurable items using:

```bash
minikube config --help
```

To set configuration properties:

```bash
minikube config set <property> <value>
```

To get current configuration:

```bash
minikube config view
```

## Minikube Add-Ons 

List addons using:

```bash
minkube addons list
```

Enable addons

```bash
minikube addon enable <addon>
```

## Set Minikube $HOME 

```bash
export MINIKUBE_HOME=/path/to/minikube/home
```

## Preferenced Setup

```bash
export MINIKUBE_HOME=/path/to/minikube/home
minikube config set cpus 8
minikube config set memory 16384

minikube start
alias kubectl="minikube kubectl --"
```
