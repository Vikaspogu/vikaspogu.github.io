---
title: "Heketi JWT token expired error"
date: 2019-07-16
description: "Troubleshooting JWT token expiration errors in Heketi on OpenShift by synchronizing NTP clocks."
tags: ["Heketi", "Gluster"]
---

> **Note:** This post is archived. Heketi and GlusterFS have been deprecated in favor of modern storage solutions like Rook-Ceph, OpenEBS, or cloud-native storage providers.

Recently I encountered a `JWT token expired` error on the heketi pod in the OpenShift cluster.

```bash
[jwt] ERROR 2019/07/16 19:17:14 heketi/middleware/jwt.go:66:middleware.(*HeketiJwtClaims).Valid: exp validation failed: Token is expired by 1h48m59s
```

After a lot of google searches, I synchronized clocks across the pod running heketi and the master nodes, which solved the issue

```bash
ntpdate -q 0.rhel.pool.ntp.org; systemctl restart ntpd
```
