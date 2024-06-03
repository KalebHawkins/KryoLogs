---
weight: 999
title: "Machinesets"
description: ""
icon: "article"
date: "2024-05-30T14:49:58-05:00"
lastmod: "2024-05-30T14:49:58-05:00"
draft: false
toc: true
---

In this document we are going to review the process of modifying a machineset to increase resources, for this example we will be increasing memory.

## Modify The Machineset

In the first step we modify the machine set we want to modify. First we decide which machineset we want to modify we can get the machinesets using the following.

```bash
oc get machineset -n openshift-machine-api
```

Now we want to modify our machineset's memory. In our case our memory is set to `memoryMib: 64000` (64GB) we want to set our memory to 128GB.

```bash
oc edit machineset <machineset> -n openshift-machine-api


# spec:
#     providerSpec:
#         ...
#         memoryMiB: 128000 # <- This was 64000
#         ...
```

## Applying The Changes

Now that our changes are made we want apply them. This isn't something that is automatically done. There are really two different methods that can be used for applying these changes.

### Method One - Out with the Old

The first method is the simplest and most straight forward. We just delete a machine managed by the machineset so another machine will be created in its place.

```bash
oc get machine -n openshift-machine-api
oc delete machine <machine>

# Wait for the new machine to provision
watch oc get machine -n openshift-machine-api
# Wait for the new node to come into a ready state
watch oc get nodes
```

This will take some time to delete and after a few minutes you should see another machine being spun up in its place. If the new machine is taking longer then 10-15 minutes to complete provisioning and the node has either not appeared or come into a ready state you can check the `machine-controller` container logs.

```bash
oc logs -c machine-controller <machine-api-controller_pod_name> -n openshift-machine-api 
```

### Method Two - Conservative

The second method is a little more conservative. First we will scale up our machineset, set an annotation on the machine we want to remove, and finally we scale back down our machineset. For this example imagine we have 3 machines in the machineset and thats what we want to end off with.

```bash
# Scale up the machineset by 1
oc scale --replicas=4 machineset <machineset> -n openshift-machine-api
# Wait for the new machine to provision
watch oc get machine -n openshift-machine-api
# Wait for the new node to come into a ready state
watch oc get nodes

# Annotation the machine you want to have removed
oc annotate machine/<machine-name> -n openshift-machine-api machine.openshift.io/cluster-api-delete-machine="true"
# Scale back down
oc scale --replicas=3 machineset <machineset> -n openshift-machine-api
```

> You can repeat either of the two methods on your machinesets for ***worker*** and ***infrastructure*** nodes.

## Master Nodes and Machinesets

> If these steps are not followed precisely it will break your cluster. You will have to rebuild the cluster there is no recovery options. 

Master nodes are a little more delegate when it comes to scaling up and down. There are additional checkout steps and those steps must be performed in order.

Step one is to verify your current cluster is healthy. There are three steps to this.

1. Check that all master nodes are available.
2. Make sure your pods are running.
3. Check etcd memberlist, status, and endpoint health.

```bash
# Check that all master nodes are available
oc get etcd -o=jsonpath='{range .items[0].status.conditions[?(@.type=="EtcdMembersAvailable")]}{.message}{"\n"}'

# Make sure your pods are running
oc -n openshift-etcd get pods -l k8s-app=etcd

# Check etcd memberlist, status, and endpoint health. <pod-name> is one of the pods from the output of the previous command
oc rsh -n openshift-etcd <pod-name>

# Check the member list
etcdctl member list -w table

# Check the status
etcdctl endpoint status -w table

# Check the health
etcdctl endpoint health -w table
```

At this point assuming everything is healthy you can scale your machineset up. 

```bash
oc scale --replicas=2 machineset <machineset> -n openshift-machine-api
```

This will take several minutes once the node is in a ready state we must wait for additional tasks to complete.

1. Node must be in a ready state.
2. Verify all cluster operators are finished progressing. 
3. Etcd pods must start up.
4. New etcd peer must join the cluster.
5. Check the status the same way we did in the first step.

```bash
oc get nodes

# All cluster operators should complete their progression before moving to the next step.
watch oc get co 

oc -n openshift-etcd get pods -l k8s-app=etcd

oc rsh -n openshift-etcd <pod-name>
etcdctl member list -w table
etcdctl endpoint status -w table
etcdctl endpoint health -w table
```

Now that we have our new master node in place it is time to remove the old one.

We need to annotate the machine we want to remove. This will tell the machine api which node to delete when scaling down. Finally, we scale down the machineset.

```bash
oc annotate machine/<machine-name> -n openshift-machine-api machine.openshift.io/cluster-api-delete-machine="true"
oc scale --replicas=1 machineset <machineset> -n openshift-machine-api
```

Provide the cluster about 10 minutes to perform and cluster operator operations. After this will still see some unhealthy statuses. This is because we must remove the etcd peer manually. To do that we need to get back into one of the etcd pods.

```bash
oc -n openshift-etcd get pods -l k8s-app=etcd

oc rsh -n openshift-etcd <pod-name>
etcdctl member list -w table
```

In the member list you will get the `ID` of the server you want to remove and perform the following command. 

```bash
etcdctl member remove <ID>
```

You will repeat the steps above for each master node machineset that you need to modify. 