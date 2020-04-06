---
layout: post
title: Taking Snapshots of AKS Persistent Volumes
author: yufufi
categories: [programming, tech]
tags: [kubernetes, AKS, devops, azure]
---

I ended up writing couple of scripts this weekend to manage snapshots of Azure Disks attached to AKS Nodes. First script goes through all the`PersistentVolume` objects in AKS (default context, all namespaces) and tries to take an incremental snapshot. A new snapshot is created every Monday, otherwise an incremental backup is taken on the existing snapshot. Second script goes through all snapshots in a resource group and deletes the ones older than the expiry date. They depend on `bash`, `jq`, `kubectl`, and `az`.

{% gist bd31c89e1a2cdfe20bd770600d22fce9 %}
