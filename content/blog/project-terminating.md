---
title: "Deleting an OpenShift Project Stuck in Terminating State"
date: 2019-05-15
description: "Fix OpenShift namespaces stuck in terminating state by removing finalizers from the project specification."
tags: ["OpenShift", "Kubernetes", "Troubleshooting"]
slug: "openshift-project-terminating"
---

Recently I faced an issue where one of my projects got stuck in a terminating state for days. The workaround below fixed the problem.

Export OpenShift project as a JSON Object

```bash
oc get project delete-me -o json > ns-without-finalizers.json
```

Replace below from

```yaml
spec:
  finalizers:
    - kubernetes
```

to

```yaml
spec:
  finalizers: []
```

On one of the master nodes, execute these commands.

```bash
kubectl proxy & PID=$!
curl -X PUT http://localhost:8001/api/v1/namespaces/delete-me/finalize \
-H "Content-Type: application/json" --data-binary @ns-without-finalizers.json
kill $PID
```
