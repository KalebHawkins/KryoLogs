---
title: Openshift - Working with vSphere Storage Classes
date: 2023-04-14
author: Kaleb Hawkins
description: Modifing and setting datastores on storage classes.
tags:
  - openshift
  - storageclass
---

In this post we are going to look at two different things. First, we will look at modifing the default datastore for our vsphere storage class. Second, we will look at creating storage classes that specify the datastore. 

## The Default Datastore

The default datastore is located in a configmap. This configmap is called `cloud-conf` it could also be called `cloud-provider-config` depending on the Openshift version. Either of these config maps can be found in the `openshift-cloud-controller-manager` namespace. We can view it using the following command.

```bash
oc get cm -n openshift-cloud-controller-manager

# Replace cloud-conf with cloud-provider-config depending on your results below.
oc get cm cloud-conf -n openshift-cloud-controller-manager -o yaml
```

You can then modify the `default-datastore` entry by editing the yaml.

```bash
oc edit cm cloud-conf -n openshift-cloud-controller-manager
```

## Specify Datastores at the StorageClass Level

You will need to create a `custom resource definition (CRD)` to create the storageclass and apply it to the cluster. The CRD is just a yaml file.

```yaml
# filename: sc-custom-ds.yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: <storage-class-name>
  annotations:
    kubernetes.io/description: A good description for your storageclass 
    storageclass.kubernetes.io/is-default-class: 'true'
provisioner: kubernetes.io/vsphere-volume
parameters:
  datastore: <datastore-name> # <- Here is where you specify your datastore.
  diskformat: eagerzeroedthick
reclaimPolicy: Retain # <- Modify this depending on your policies (see references for more info)
volumeBindingMode: Immediate
```

Now you can apply the CRD to the cluster using whatever method you use for cluster config, gitops, etc. For this demo I'll use `kubectl`.

```bash
kubectl apply -f sc-custom-ds.yaml
```

That is all there is to it. Now, when you create your `PVs` against the specified storage class they will use the datastore provided in the configuration. 

## References

- [Default Datastore](https://access.redhat.com/solutions/4618011)
- [Reclaim Policies](https://kubernetes.io/docs/concepts/storage/storage-classes/)