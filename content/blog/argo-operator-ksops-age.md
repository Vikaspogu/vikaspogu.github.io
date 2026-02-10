---
title: "Why I Switched from FluxCD to ArgoCD (And Almost Switched Back)"
date: 2022-08-10
description: "The real story of migrating GitOps secrets management from FluxCD's native SOPS to ArgoCD with KSOPS—including what broke and why."
tags: ["ArgoCD", "GitOps", "SOPS", "Secrets", "Kubernetes"]
slug: "argocd-ksops-age"
---

## The Problem I Was Actually Solving

I'd been running FluxCD for two years. Secrets management was seamless—FluxCD has native SOPS integration, and it just worked. But my team standardized on ArgoCD, and I needed to migrate.

What I thought would be a weekend project turned into a week of debugging. Here's what actually happened.

## Why Not Just Use Sealed Secrets or External Secrets?

Before diving into SOPS, I evaluated the alternatives:

**Sealed Secrets:** Requires the controller to be running. If you're bootstrapping a cluster from scratch, you have a chicken-and-egg problem. I wanted secrets in git that work even for cluster bootstrapping.

**External Secrets Operator:** Great if you're already invested in Vault or AWS Secrets Manager. I wasn't, and I didn't want to add another dependency for a home lab.

**SOPS:** Secrets encrypted in git, decrypted at apply time. No external dependencies. This is what I wanted.

## What I Tried First (And Why It Failed)

### Attempt 1: Following the KSOPS README

The [KSOPS documentation](https://github.com/viaduct-ai/kustomize-sops#argo-cd-integration-) shows a clean integration. I copied it verbatim.

**Result:** ArgoCD showed `Unknown` sync status. No errors in the UI, just... nothing.

**The actual problem:** The docs assume you're running vanilla ArgoCD. I was using the OpenShift GitOps Operator, which runs the repo-server with a non-root user. The KSOPS binary couldn't execute because of permission issues that never surfaced as clear errors.

### Attempt 2: Using PGP Instead of Age

I initially tried PGP because that's what most tutorials showed. Generating keys, importing them, managing keyrings—it was a mess.

**The actual problem:** PGP key management is painful. SOPS now recommends Age encryption, which uses simple file-based keys. I wasted a day before discovering this.

## The Solution That Actually Works on OpenShift

Here's the configuration that works with the OpenShift GitOps Operator. The key differences from the standard docs:

### 1. Age Over PGP

Generate an Age key (this is the entire "key management" process):

```bash
age-keygen -o age.agekey
# Public key: age1helqcqsh9464r8chnwc2fzj8uv7vr5ntnsft0tn45v2xtz0hpfwq98cmsg
```

Store it as a secret:

```bash
cat age.agekey | kubectl create secret generic sops-age \
  --namespace=openshift-gitops \
  --from-file=key.txt=/dev/stdin
```

### 2. The ArgoCD CR Configuration

This is the part that differs from most tutorials. Note the specific paths—these matter on OpenShift:

```yaml
repo:
  env:
    - name: XDG_CONFIG_HOME
      value: /.config
    - name: SOPS_AGE_KEY_FILE
      value: /.config/sops/age/keys.txt
  volumes:
    - name: custom-tools
      emptyDir: {}
    - name: sops-age
      secret:
        secretName: sops-age
  initContainers:
    - name: install-ksops
      image: viaductoss/ksops:v3.0.2
      command: ["/bin/sh", "-c"]
      args:
        - 'echo "Installing KSOPS..."; cp ksops /custom-tools/; cp $GOPATH/bin/kustomize /custom-tools/; echo "Done.";'
      volumeMounts:
        - mountPath: /custom-tools
          name: custom-tools
  volumeMounts:
    - mountPath: /usr/local/bin/kustomize
      name: custom-tools
      subPath: kustomize
    - mountPath: /.config/kustomize/plugin/viaduct.ai/v1/ksops/ksops
      name: custom-tools
      subPath: ksops
    - mountPath: /.config/sops/age/keys.txt
      name: sops-age
      subPath: keys.txt
```

**Why `XDG_CONFIG_HOME`?** Kustomize looks for plugins in `$XDG_CONFIG_HOME/kustomize/plugin/`. Without this, it defaults to `~/.config`, which doesn't exist in the container.

**Why the specific mount paths?** KSOPS must be at the exact path Kustomize expects for the plugin API version (`viaduct.ai/v1/ksops`). Get this wrong and you'll get silent failures.

## What I'd Do Differently

1. **Start with Age, not PGP.** The documentation ecosystem hasn't caught up, but Age is simpler in every way.

2. **Test locally first.** Before touching ArgoCD, verify your SOPS + Kustomize setup works with `kustomize build` on your laptop.

3. **Check the repo-server logs.** The ArgoCD UI hides most errors. The real debugging happens in `kubectl logs -n openshift-gitops deployment/openshift-gitops-repo-server`.

4. **Consider if you need this complexity.** For a home lab, this works great. For production with a team, External Secrets Operator with a proper secrets manager might be worth the additional dependency.

## The Takeaway

GitOps secrets management isn't a solved problem—it's a series of tradeoffs. SOPS gives you encrypted secrets in git with no external dependencies, but the ArgoCD integration requires understanding how Kustomize plugins work and where ArgoCD expects to find them.

The official docs get you 80% there. The last 20% depends on your specific ArgoCD deployment method, and that's where most people get stuck.
