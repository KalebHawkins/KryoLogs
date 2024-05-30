---
weight: 999
title: "Authentication Operator Degraded"
description: ""
icon: "article"
date: "2024-05-30T14:49:49-05:00"
lastmod: "2024-05-30T14:49:49-05:00"
draft: false
toc: true
---

## Issue Summary

When performing an `oc get co` the authentication cluster operator is reporting a degraded state with the following message.

```bash
OAuthServerRouteEndpointAccessibleControllerAvailable: Get "https://oauth-openshift.apps.zone.domain.com/healthz": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
```

## Root Cause

This can be a result of many different things, network issues, domain name resolution issues, etc. In my case the authentication pods and authentication-operator pod running in the cluster were failing to reconciliate due to some node modifications.

## Resolution

Delete the authentication pods and authentication operator pods.

```bash
# Get the authentication pod names
oc get pods -n openshift-authentication

# Delete the pods (repeat this step for each pod)
oc delete pods -n openshift-authentication <podName>
```

```bash
# Get the authentication-operator pod names
oc get pods -n openshift-authentication-operator

# Delete the operator pod
oc delete pods -n openshift-authentication-operator <podName>
```