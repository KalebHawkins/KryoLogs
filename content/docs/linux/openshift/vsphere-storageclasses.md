---
weight: 999
title: "VSphere StorageClasses"
description: ""
icon: "article"
date: "2024-05-30T14:50:23-05:00"
lastmod: "2024-05-30T14:50:23-05:00"
draft: false
toc: true
---

In this post we are going to look at two different things. First, we will look at modifying the default datastore for our vsphere storage class. Second, we will look at creating storage classes that specify the datastore. 

## The Default Datastore

> This change will trigger the nodes to reboot. Always make sure you are in an open maintenance window or that your applications are designed to work through pods migrating from node to node for less impact.

The default datastore is located in a configmap. This configmap is called `cloud-provider-config`. This configmap can be found in the  `openshift-config` namespace. We can view it using the following command.

```bash
oc get cm cloud-provider-config -n openshift-config -o yaml
```

You can then modify the `default-datastore` entry by editing the yaml.

```bash
oc edit cm cloud-provider-config -n openshift-config
```

This will trigger the `kube-controller-manager` cluster operator to start updating which will cause the cluster nodes to reboot. This will cause other cluster operators to update as the master nodes roll over. This can take variable amount of time to complete depending on the size of the cluster. As long as everything is pprogressing you are OK 😬. You can watch the cluster update using the command below. You may have to reauthenticate at some point but again all should be fine. There is little to no application impact. The only impact is when the pods are moving from node to node and depending on the application design this may or may not be an issue.

```bash
watch oc get co
```

Once all of that is done you can verify the change by looking at the `cloud-conf` configmap in the `openshift-cloud-controller-manager` namespace.

```bash
oc get cm -n openshift-cloud-controller-manager -o yaml
```

Test provisioning of a new PV as well. Make sure it get placed in the right place. Make your `custom resource definition (CRD)` and apply it to the cluster using whatever method you use for cluster or application config, gitops, etc. For this demo I'll use `kubectl`.

```yaml
# filename: new-pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: <name-of-pvc>
  namespace: <name-of-namespace>
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: <storageclass-name>
  volumeMode: Filesystem
```

```bash
kubectl apply -f new-pvc.yaml
```

## Specify Datastores at the StorageClass Level

You will need to create a CRD to create the storageclass and apply it to the cluster. The CRD is just a yaml file.

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

Now you can apply the CRD to the cluster.

```bash
kubectl apply -f sc-custom-ds.yaml
```

That is all there is to it. Now, when you create your `PVs` against the specified storage class they will use the datastore provided in the configuration. 

## References

- [Default Datastore](https://access.redhat.com/solutions/4618011)
- [Reclaim Policies](https://kubernetes.io/docs/concepts/storage/storage-classes/)