+++ 
date = 2019-12-18
title = "Permission denied pushing to OpenShift Registry"
description = "Permission denied on pushing build images into OpenShift internal registry"
slug = "" 
tags = ["OpenShift"]
categories = []
externalLink = ""
series = []
socialShare=true
+++

Recently, I ran into an issue where pushing images to the docker registry after a build fails.

```bash
Pushing image docker-registry.default.svc:5000/simple-go-build/simple-go:latest ...
Registry server Address:
Registry server User Name: serviceaccount
Registry server Email: serviceaccount@example.org
Registry server Password: <<non-empty>>
error: build error: Failed to push image: received unexpected HTTP status: 500 Internal Server Error
```

Registry pods logs show `permission denied`.

```bash
err.code=UNKNOWN err.detail="filesystem: mkdir /registry/docker/registry/v2/repositories/simple-go-build/simple-go/_uploads/c34415b4-c6d8-42ba-9854-aee449efd984: permission denied"
```

One of the Red Hat solutions articles suggested verifying the file ownership of the files and directories in the volume and comparing it to the uid of the registry.

Changing the owner recursively to the uid of the registry fixed the issue

```bash
root@master# chown -R 1001 /exports/registry/docker/
```
