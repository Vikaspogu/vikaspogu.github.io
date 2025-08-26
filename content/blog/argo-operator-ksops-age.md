---
title: "ArgoCD with Kustomize and KSOPS using Age encryption"
date: "2022-08-10T16:31:43-05:00"
comments: false
socialShare: true
toc: false
tags: ["ArgoCD", "gitops", "sops"]
---

I am a big fan of FluxCD and its integration with secrets management; however, recently, I decided to tinker with ArgoCD. One of the challenges I encountered was secrets management. Although ArgoCD provides flexible integration with most secrets management tools, it requires little extra configuration.

So I started my journey to configure ArgoCD with SOPS; there isn't a direct integration with SOPS. So we will have to use the [Kustomize](https://github.com/kubernetes-sigs/kustomize/) plugin called [ksops](https://github.com/viaduct-ai/kustomize-sops), which is the suggested tool on Argo's website and it has pretty good instructions for [Argo integration](https://github.com/viaduct-ai/kustomize-sops#argo-cd-integration-).

## What is KSOPS?

KSOPS, or kustomize-SOPS, is a kustomize plugin for managing SOPS encrypted resources. KSOPS is used to decrypt any Kubernetes resources but is most commonly used to decrypt encrypted Kubernetes Secrets and ConfigMaps. The main goal of KSOPS is to manage encrypted resources the same way we manage the Kubernetes manifests.

## Setup

I am setting up this integration on my OpenShift cluster and deploying ArgoCD via [Operator](https://argocd-operator.readthedocs.io/en/latest/). For file encryption, I'll use [Age encryption](https://github.com/FiloSottile/age) tool recommended over PGP by SOPS documentation.

### Generate age key

[Install](https://github.com/FiloSottile/age#installation) age package

Generating an age key using age-keygen:

```bash
$ age-keygen -o age.agekey
Public key: age1helqcqsh9464r8chnwc2fzj8uv7vr5ntnsft0tn45v2xtz0hpfwq98cmsg
Create a secret with the age private key; the key name must end with .agekey to be detected as an age key:
```

### Secret

Create a kubernetes secret from the age key in `openshift-gitops` project

```bash
$ cat age.agekey | kubectl create secret generic sops-age --namespace=openshift-gitops \
--from-file=key.txt=/dev/stdin
```

### Configuration

Update the `spec.repo`` definition of ArgoCD CR to configure KSOPS custom tooling. Below configuration mounts the age secret and installs the KSOPS tool using initContainer

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

Here is a [guide](https://github.com/viaduct-ai/kustomize-sops#3-configure-sops-via-sopsyaml) for creating and testing the resources
