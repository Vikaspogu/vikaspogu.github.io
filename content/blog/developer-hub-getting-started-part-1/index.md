---
title: "Getting Started with Red Hat Developer Hub - Part 1"
date: 2024-01-13
description: "Deploy Red Hat Developer Hub (Backstage) on OpenShift with Helm, configure plugins for GitHub, ArgoCD, and Open Cluster Management."
tags: ["Developer Hub", "Backstage", "IDP", "OpenShift"]
slug: "developer-hub-getting-started-part-1"
series: ["Developer Hub"]
---

## Why I Wanted an Internal Developer Portal

Every platform team eventually faces the same question: how do developers discover what's available? We had ArgoCD for deployments, Open Cluster Management for multi-cluster visibility, and GitHub for code—but no single place to see it all.

I'd heard about Backstage for years but never had time to evaluate it. When Red Hat released Developer Hub (their supported Backstage distribution), I decided to try building a self-service portal where developers could provision databases without filing tickets.

This post covers what I learned deploying it—including the configuration gotchas that aren't obvious from the docs.

## Choosing Red Hat Developer Hub Over Vanilla Backstage

**Why not just run Backstage directly?**

I tried. Backstage requires you to build and maintain your own container image with plugins baked in. Every plugin update means rebuilding. For a team with time to invest, this is fine. For a quick evaluation, it's friction.

Developer Hub ships pre-built with dynamic plugins—you enable them via config, not code. The tradeoff is less flexibility, but I wanted to validate the concept before committing to custom development.

## Installation: What the Docs Don't Emphasize

The Helm install itself is straightforward:

```bash
helm repo add openshift-helm-charts https://charts.openshift.io/
helm show values openshift-helm-charts/redhat-developer-hub > values.yaml
```

Update the `global.clusterRouterBase` according to OpenShift router host and modify values if needed

```yaml {linenos=table,hl_lines=7}
global:
  auth:
    backend:
      enabled: true
      existingSecret: ""
      value: ""
  clusterRouterBase: apps.example.com
...
```

Install the helm chart with modified values

```bash
oc new-project developer-hub
helm install developer-hub developer-hub/developer-hub -f values.yaml
```

Wait for the developer hub and postgresql database pods to be in ready status

```bash
oc get pods
NAME                             READY   STATUS    RESTARTS   AGE
developer-hub-64d8cff99c-k8k9r   1/1     Running   0          2d
developer-hub-postgresql-0       1/1     Running   0          2d
```

Navigate to the developer hub in a web browser by grabbing the route host information as shown below

```bash
oc get routes
NAME            HOST/PORT                                         PATH   SERVICES        PORT           TERMINATION     WILDCARD
developer-hub   developer-hub-developer-hub.apps.example.com   /      developer-hub   http-backend   edge/Redirect   None
```

At this point you have a running Developer Hub—but it's useless. It's an empty portal with no integrations. The real work starts now.

## The Plugin Problem I Didn't Anticipate

Backstage's value comes from plugins. Developer Hub ships with many pre-installed but disabled. My goal was to integrate:

- **GitHub** for authentication (so developers use their existing credentials)
- **Kubernetes** plugin to show workloads
- **Open Cluster Management** to display all managed clusters
- **ArgoCD** to show deployment sync status

**What I learned:** Enabling plugins is easy. Configuring them to actually work together is where you'll spend your time. Each plugin needs its own authentication, and the documentation assumes you're doing one at a time.

### Enable Plugins

Developer Hub images come with [dynamic plugins](https://access.redhat.com/documentation/en-us/red_hat_developer_hub/1.0/html/administration_guide_for_red_hat_developer_hub/rhdh-installing-dynamic-plugins#rhdh-supported-plugins) pre-installed but disabled. Enable them in your Helm values:

> **Gotcha:** The `package` paths must match exactly what's in the image. These paths change between versions. If you upgrade Developer Hub and plugins stop working, check if the paths changed.

```yaml {linenos=table,hl_lines="11-33"}
global:
  auth:
    backend:
      enabled: true
      existingSecret: ""
      value: ""
  clusterRouterBase: apps.example.com
  dynamic:
    includes:
      - dynamic-plugins.default.yaml
    plugins:
      - disabled: false
        package: ./dynamic-plugins/dist/backstage-plugin-catalog-backend-module-github-dynamic
      - disabled: false
        package: ./dynamic-plugins/dist/roadiehq-backstage-plugin-security-insights
      - disabled: false
        package: ./dynamic-plugins/dist/backstage-plugin-kubernetes-backend-dynamic
      - disabled: false
        package: ./dynamic-plugins/dist/backstage-plugin-kubernetes
      - disabled: false
        package: ./dynamic-plugins/dist/roadiehq-backstage-plugin-argo-cd-backend-dynamic
      - disabled: false
        package: ./dynamic-plugins/dist/roadiehq-scaffolder-backend-argocd-dynamic
      - disabled: false
        package: ./dynamic-plugins/dist/roadiehq-backstage-plugin-argo-cd
      - disabled: false
        package: ./dynamic-plugins/dist/janus-idp-backstage-plugin-ocm
      - disabled: false
        package: ./dynamic-plugins/dist/janus-idp-backstage-plugin-ocm-backend-dynamic
      - disabled: false
        package: ./dynamic-plugins/dist/roadiehq-scaffolder-backend-module-utils-dynamic
...
```

Re-install the chart with updated values

```bash
helm upgrade --install developer-hub developer-hub/developer-hub -f values.yaml
```

Verify if plugins are installed in the `install-dynamic-plugins init container` within the Developer Hub pod’s log

```log
======= Skipping disabled dynamic plugin ./dynamic-plugins/dist/backstage-plugin-scaffolder-backend-module-gitlab-dynamic
======= Installing dynamic plugin ./dynamic-plugins/dist/backstage-plugin-kubernetes-backend-dynamic
	==> Grabbing package archive through `npm pack`
	==> Removing previous plugin directory /dynamic-plugins-root/backstage-plugin-kubernetes-backend-dynamic-0.13.0
	==> Extracting package archive /dynamic-plugins-root/backstage-plugin-kubernetes-backend-dynamic-0.13.0.tgz
	==> Removing package archive /dynamic-plugins-root/backstage-plugin-kubernetes-backend-dynamic-0.13.0.tgz
	==> Merging plugin-specific configuration
	==> Successfully installed dynamic plugin /opt/app-root/src/dynamic-plugins/dist/backstage-plugin-kubernetes-backend-dynamic
======= Installing dynamic plugin ./dynamic-plugins/dist/backstage-plugin-kubernetes
	==> Grabbing package archive through `npm pack`
	==> Removing previous plugin directory /dynamic-plugins-root/backstage-plugin-kubernetes-0.11.0
	==> Extracting package archive /dynamic-plugins-root/backstage-plugin-kubernetes-0.11.0.tgz
	==> Removing package archive /dynamic-plugins-root/backstage-plugin-kubernetes-0.11.0.tgz
	==> Merging plugin-specific configuration
	==> Successfully installed dynamic plugin /opt/app-root/src/dynamic-plugins/dist/backstage-plugin-kubernetes
======= Installing dynamic plugin ./dynamic-plugins/dist/janus-idp-backstage-plugin-topology
```

## Plugin Integrations

> In the spirit of brevity and to ensure accuracy, I've posted the configuration code snippets for your convenience. However, for a more detailed and nuanced understanding of the integration process, I highly recommend referring to the official product documentation.

> [!IMPORTANT]
> Integration code snippets are added to the configmap; for reference, here is the complete [configmap](https://raw.githubusercontent.com/Vikaspogu/openshift-multicluster/main/kustomize/cluster-overlays/pxm-acm/developer-hub-chart/app-config-rhdh.yaml)

### Externalize configuration

Create a `configmap` to add configuration to the developer hub as described in the [documentation](https://access.redhat.com/documentation/en-us/red_hat_developer_hub/1.0/html/getting_started_with_red_hat_developer_hub/ref-rhdh-supported-configs_rhdh-getting-started)

![Alt text](configmap.png "ConfigMap")

Mount the ConfigMap, add `extraAppConfig` to the values file under `upstream.backstage` section

```yaml {linenos=table,hl_lines="21-23"}
upstream:
  backstage:
    appConfig:
      app:
        baseUrl: 'https://{{- include "janus-idp.hostname" . }}'
      backend:
        baseUrl: 'https://{{- include "janus-idp.hostname" . }}'
        cors:
          origin: 'https://{{- include "janus-idp.hostname" . }}'
        database:
          connection:
            password: ${POSTGRESQL_ADMIN_PASSWORD}
            user: postgres
        auth:
          keys:
            - secret: ${BACKEND_SECRET}
    args:
      - "--config"
      - dynamic-plugins-root/app-config.dynamic-plugins.yaml
    command: []
    extraAppConfig:
      - configMapRef: app-config-rhdh
        filename: app-config-rhdh.yaml
```

Re-install the chart with updated values

```bash
helm upgrade --install developer-hub developer-hub/developer-hub -f values.yaml
```

### GitHub Integration

Seamlessly connect your projects with GitHub by configuring the [integration](https://access.redhat.com/documentation/en-us/red_hat_developer_hub/1.0/html/getting_started_with_red_hat_developer_hub/ref-rhdh-supported-configs_rhdh-getting-started#setting-github-integration-and-authentication).

Create a secret named `rhdh-secrets` as mentioned in documentation with GitHub clientId, secret and token

```bash
oc create secret generic rhdh-secrets --from-literal=GITHUB_APP_CLIENT_ID=dummy --from-literal=GITHUB_APP_CLIENT_SECRET=dummy --from-literal=GITHUB_TOKEN=dummy
```

Mount the `rhdh-secrets`, add `extraEnvVarsSecrets` to the values file under `upstream.backstage` section

```yaml {linenos=table,hl_lines="21-22"}
upstream:
  backstage:
    appConfig:
      app:
        baseUrl: 'https://{{- include "janus-idp.hostname" . }}'
      backend:
        baseUrl: 'https://{{- include "janus-idp.hostname" . }}'
        cors:
          origin: 'https://{{- include "janus-idp.hostname" . }}'
        database:
          connection:
            password: ${POSTGRESQL_ADMIN_PASSWORD}
            user: postgres
        auth:
          keys:
            - secret: ${BACKEND_SECRET}
    args:
      - "--config"
      - dynamic-plugins-root/app-config.dynamic-plugins.yaml
    command: []
    extraEnvVarsSecrets:
      - rhdh-secrets
    extraAppConfig:
      - configMapRef: app-config-rhdh
        filename: app-config-rhdh.yaml
```

Update the `app-config-rhdh` configmap and add below configuration

```yaml
integrations:
  github:
    - host: github.com
      token: ${GITHUB_TOKEN}
auth:
  # see https://backstage.io/docs/auth/ to learn about auth providers
  environment: development
  providers:
    github:
      development:
        clientId: ${GITHUB_APP_CLIENT_ID}
        clientSecret: ${GITHUB_APP_CLIENT_SECRET}
```

Re-installing the chart will recreate a new pod with updated values and configmap

```bash
helm upgrade --install developer-hub developer-hub/developer-hub -f values.yaml
```

![Alt text](github-auth.png "GitHub Auth")

### Navigating Clusters with Open Cluster Management

Open Cluster Management extends our reach to manage and monitor multiple clusters effortlessly. To enable we can add kubernetes [configuration](https://backstage.io/docs/features/kubernetes/configuration) and reference the cluster in catalog provider

We are setting up authentication for the Kubernetes cluster using the serviceaccount token. Create a serviceaccount in the developer-hub namespace

```bash
oc create sa developer-hub -n developer-hub
```

Obtain a long-lived token by creating a secret

```bash
oc apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: developer-hub-sa
  namespace: developer-hub
  annotations:
    kubernetes.io/service-account.name: developer-hub
type: kubernetes.io/service-account-token
EOF
```

Add a new entry in the `extraEnvVars` to mount the serviceaccount token secret as an environment variables in the values file under `upstream.backstage` section

```yaml {linenos=table,hl_lines="20-24"}
upstream:
  backstage:
    ...
    extraEnvVarsSecrets:
      - rhdh-secrets
    extraAppConfig:
      - configMapRef: app-config-rhdh
        filename: app-config-rhdh.yaml
    extraEnvVars:
      - name: BACKEND_SECRET
        valueFrom:
          secretKeyRef:
            key: backend-secret
            name: '{{ include "janus-idp.backend-secret-name" $ }}'
      - name: POSTGRESQL_ADMIN_PASSWORD
        valueFrom:
          secretKeyRef:
            key: postgres-password
            name: '{{- include "janus-idp.postgresql.secretName" . }}'
      - name: KUBE_TOKEN
        valueFrom:
          secretKeyRef:
            key: token
            name: developer-hub-sa
```

Update the `app-config-rhdh` configmap with below configuration

```yaml
kubernetes:
  serviceLocatorMethod:
    type: "multiTenant"
  clusterLocatorMethods:
    - type: "config"
      clusters:
        - url: https://api.example.com:6443
          name: acm
          authProvider: "serviceAccount"
          skipTLSVerify: true
          serviceAccountToken: ${KUBE_TOKEN}
          dashboardApp: openshift
          dashboardUrl: https://console-openshift-console.apps.example.com/
catalog:
  providers:
    ocm:
      default:
        kubernetesPluginRef: acm #same as the clusters name in kubernetes section
        name: multiclusterhub
        owner: group:ops
        schedule:
          frequency:
            seconds: 10
          timeout:
            seconds: 60
```

Re-installing the chart will recreate a new pod with updated values and mount serviceaccount token secret as an environment variable

```bash
helm upgrade --install developer-hub developer-hub/developer-hub -f values.yaml
```

![Alt text](ocm.png "Multicluster")

### Synchronizing with ArgoCD

The ArgoCD Backstage plugin provides synced, health status and updates the history of your services to your Developer Portal.

Generate a new token from the ArgoCD instance

![Alt text](argocd-token.png "ArgoCD token")

Create a new secret from the ArgoCD token

```bash
oc create secret generic argocd-token --from-literal=ARGOCD_TOKEN=dummy
```

Add a new entry in the `extraEnvVars` to mount the secret as an environment variables in the values file under `upstream.backstage` section

```yaml {linenos=table,hl_lines="25-29"}
upstream:
  backstage:
    ...
    extraEnvVarsSecrets:
      - rhdh-secrets
    extraAppConfig:
      - configMapRef: app-config-rhdh
        filename: app-config-rhdh.yaml
    extraEnvVars:
      - name: BACKEND_SECRET
        valueFrom:
          secretKeyRef:
            key: backend-secret
            name: '{{ include "janus-idp.backend-secret-name" $ }}'
      - name: POSTGRESQL_ADMIN_PASSWORD
        valueFrom:
          secretKeyRef:
            key: postgres-password
            name: '{{- include "janus-idp.postgresql.secretName" . }}'
      - name: KUBE_TOKEN
        valueFrom:
          secretKeyRef:
            key: token
            name: developer-hub-sa
      - name: ARGOCD_TOKEN
        valueFrom:
          secretKeyRef:
            key: ARGOCD_TOKEN
            name: argocd-token
```

Update the `app-config-rhdh` configmap with below configuration

```yaml
argocd:
  appLocatorMethods:
    - instances:
        - name: main
          token: ${ARGOCD_TOKEN}
          url: https://openshift-gitops-server-openshift-gitops.apps.example.com
      type: config
```

Re-installing the chart

```bash
helm upgrade --install developer-hub developer-hub/developer-hub -f values.yaml
```

![Alt text](argocd.png "ArgoCD")

## What I Learned Setting This Up

**Time investment:** The initial deployment took 30 minutes. Getting all four integrations working took two days. Most of that was understanding how secrets flow between the Helm values, ConfigMaps, and the actual Backstage config.

**The biggest gotcha:** Every integration needs its own service account or token, and they all have different permission requirements. The Kubernetes plugin needs cluster-wide read access. The ArgoCD plugin needs a token with specific RBAC. Plan for this before you start.

**Is it worth it?** For a home lab exploration, absolutely—I learned a lot about how Backstage works. For production, you need to budget significant time for ongoing maintenance. Plugins break across Backstage upgrades, and the ecosystem moves fast.

## What's Next

In [Part 2](https://vikaspogu.dev/blog/developer-hub-getting-started-part-2/), I'll cover the part that actually delivers value to developers: software templates. We'll build a self-service workflow where developers can provision a CloudNative-PG database cluster by filling out a form—no tickets, no waiting.
