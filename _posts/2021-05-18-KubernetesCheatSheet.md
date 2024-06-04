---
layout: post
title: "Kubernetes Cheat Sheet"
excerpt: "Kubectl cheat sheet for Kubernetes. It contains commands I use most often while working with K8S. It is not complete. I will keep adding commands and re-organizing sections as this list grows."
date: 2021-05-18
---

This is my **kubectl** cheat sheet for Kubernetes. It contains commands I use most often while working with K8S. It is not complete. I will keep adding commands and re-organizing sections as this list grows. I do it primarily for myself but if I am able to help at least someone else out there, I would be extremely happy.

## Overview

![](https://igorski.co/content/images/2021/05/Untitled-2021-05-19-2354-1-.png)

There is a long list of Kubernetes resource types. But most commonly you will probably need to use the following few:

| Resouce | Short Name |
| --- | --- |
| pods | right-aligned |
| nodes | no |
| deployments | deploy |
| daemonsets | ds |
| replicasets | rs |

In commands you can use either the full resource name or the short name.

### Pods

Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.

A Pod is a group of one or more containers, with shared storage and network resources, and a specification for how to run the containers. A pod always runs on a node.

### Nodes

Nodes can be physical or virtual machines. Each node runs at least a **kubelet** instance and a container runtime. The kubelet process is by the Mater node to communicate to the node. The most often used container runtime is Docker.

### Deployments

Deployments allow you to describe the lifecycle of an application like which images to use, the number of pods that should exist and the way they should be upgraded.

They help to automate the update procedures that would otherwise require many tedious and error prone manual steps.

## Kubectl commands

Kubernetes has a few basic commands for managing resources:

-   **explain**: show information about the resource
-   **get**: display one or more resources
-   **edit**: edit a resource on the server
-   **delete**: Delete resources by filenames, stdin, resources and names, or by resources and label selector

These commands you can use with any type of resource, regardless if that is pods, deployments, services or any other type.

### General Commands

```bash
#List all resources in all namespaces
kubectl get all --all-namespaces
```

### Resource Commands

In the following set of commands I use pods but you can replace pods with a different resource type, like services, and the commands would still work. The principle is the same.

```bash
kubectl get pods --show-labels

#Filter based on the given selector.
kubectl get pods --selector app!=helloworrld,env=staging

#Filter based on label.
kubectl get pods -l 'release-version not in (1.0,2.0)' --show-labels

#Get pods with output in the given format.
#The format can be: json,yaml or wide. -n is to specify the namespace.
kubectl get po helloworld -n hello -o json

#Delete pods with a certain label.
kubectl delete pods -l env=staging

# Describe the pod named helloworld in the default namespaces.
# When describing a specific pod the --all-namespaces flag won't work.
kubectl describe po helloworld -n hello -o yml
```

### Context commands

Sometimes need to work with multiple clusters. Those could be your local cluster on your development machine or different deployment environments like testing, staging or production. K8S uses contexts in order to ease the access to different clusters. A context element in a kubeconfig file is used to group access parameters under a convenient name. Kubectl will run all the commands against the currently configured context.

```bash
#List all contexts
kubectl config get-contexts

#Show the current context
kubectl config current-context

#Use a particular context
kubectl config use-context my-cluster-name
```

Contexts are configured in the **~/.kube/config** file. With two clusters, it would look something like:

```
apiVersion: v1
clusters:
- cluster:
    server: https://to.cluster.one
  name: clusterOne
- cluster:
    server: https://to.cluster.two
  name: clusterTwo
contexts:
- context:
    cluster: clusterOne
    user: clusterOne
  name: clusterOne
- context:
    cluster: clusterTwo
    user: clusterTwo
  name: clusterTwo
current-context: clusterTwo
kind: Config
preferences: {}
users:
- name: userClusterOne
  user:
    token: token-clusterOne
- name: userClusterTwo
  user:
    token: token-clusterTwo
```

### Troubleshooting commands

```bash
#Get information about resource usage by cluster nodes
kubectl top node
```

### Other Commands

```bash
kubectl label pod/helloworld app=helloworld --overwrite
kubectl expose deployment helloworld --type=NodePort

kubectl create -f file_name.yml
kubectl create -f helloworld.yml --record

kubectl set image deployment/helloworld helloworld=igorski/helloworld:latest

kubectl rollout history deployment/helloworld
kubectl rollout undo deployment/helloworld [--to-revision=]

# Identifying problems:

kubectl describe deployment helloworld
kubectl logs helloworld
kubectl exec -it helloworld /bin/bash

kubectl create namespace nameOne
kubectl get namespaces
kubectl delete namespace nameOne

kubectl create -f filename.yaml -n nameOne
```

## Links

-   Cover photo by [Vidar Nordli-Mathisen](https://unsplash.com/@vidarnm?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/sailing?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
