---
title: Setup vcsim 
date: 2023-02-10
author: Kaleb Hawkins
description: Setup a vCenter api simulator for basic testing.
tags:
  - docker
  - containers
  - vcsim
  - vmware
---

- [Overview](#overview)
- [Usage](#usage)

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
