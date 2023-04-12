---
title: Openshift - Clsuter Ownership Transfer
date: 2022-05-16
author: Kaleb Hawkins
description: Transfer Cluster Ownership
tags:
  - openshift
---

- [Overview](#overview)
- [Transfering Ownership](#transfering-ownership)
- [Update the Pull Secret](#update-the-pull-secret)
- [Verification Steps](#verification-steps)
- [Reference](#reference)

## Overview

There are two steps to transfer ownership to a new user or organization.

- [ ] Initiate the transfer in Openshift Cluster Manager.
- [ ] Change the cluster pull secret to the new owner's pull secret from CLI,

> The second step must take place within 5 days of the transfer initiation of cluster transfer from the Openshift Cluster Manager.

## Transfering Ownership

1. Login to the [Openshift Cluster Manager](https://console.redhat.com/openshift) as the current owner.
2. Select the cluster you want to transfer from the **Clusters** list.
3. Click **Actions** -> **Transfer Cluster Ownship**.
4. Click **Initiate transfer** to confirm this action.

## Update the Pull Secret

The pull secret must be updated on the cluster you are wanting to transfer ownership of.

> Clusters that are versions earlier than 4.7.4 requires reboot to update the pull secret. This can temporarily limit usability of the cluster.

1. Log into the [Openshift Cluster Manager](https://console.redhat.com/openshift) as the user taking ownership of the cluster.
2. Download or copy the pull secret from the [Downloads](https://console.redhat.com/openshift/downloads) page under **Tokens**.
3. Run the following command using the pull secret you downloaded. 

```bash
oc set data secret/pull-secret -n openshift-config --from-file=.dockerconfigjson=pull-secret.txt
```

This begins updates to all nodes in the cluster, which can take some time depending on the size of your cluster.

## Verification Steps

1. Log into the [Openshift Cluster Manager](https://console.redhat.com/openshift) as the user taking ownership of the cluster.
2. In **Details**, the **Owner** has been updated.
3. In **Cluster history**, details of the transfer appear.

## Reference

- [Red Hat Cluster Manager](https://access.redhat.com/documentation/en-us/openshift_cluster_manager/2022/html/managing_clusters/assembly-managing-clusters#transferring-cluster-ownership_downloading-and-updating-pull-secrets)