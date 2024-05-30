---
weight: 999
title: "VCSim"
description: ""
icon: "article"
date: "2024-05-30T14:57:09-05:00"
lastmod: "2024-05-30T14:57:09-05:00"
draft: false
toc: true
---

## Overview

This package implements a vSphere Web Services (SOAP) SDK endpoint intended for testing consumers of the API. While the mock framework is written in the Go language, it can be used by any language that can talk to the vSphere API.

See their [github](https://github.com/vmware/govmomi/tree/master/vcsim) for more info.

## Usage

To run `vcsim` using docker run the following: 

```bash
docker run -p 8989:8989 vmware/vcsim:latest
```

To test functionality of the container you can run the following command. 

```bash
curl -sk https://user:pass@127.0.0.1:8989/about
# Example out
#{
#  "Methods": [
#    "AcquireCloneTicket",
#    "AcquireGenericServiceTicket",
#    "AddAuthorizationRole",
#    "AddCustomFieldDef",
#    "AddDVPortgroup_Task",
#    "AddHost_Task",
```