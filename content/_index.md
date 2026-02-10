---
title: DevOps Journal
layout: hextra-home
---

<div class="hx:mt-6 hx:mb-6">
{{< hextra/hero-headline >}}
  Vikas Pogu
{{< /hextra/hero-headline >}}
</div>

<div class="hx:mb-12">
{{< hextra/hero-subtitle >}}
  Senior Engineer at NVIDIA | Infrastructure & DevOps in Omniverse
{{< /hextra/hero-subtitle >}}
</div>

<div class="hx:mb-6" style="font-size: 1.1rem; line-height: 1.8; max-width: 720px; margin: 0 auto;">
  I write about what actually works in production—not just the happy path from the docs. This journal captures failed experiments, unexpected gotchas, and the decisions behind infrastructure choices. The stuff AI can't tell you because it requires breaking things first.
</div>

<div class="hx:mb-6 hx:mt-8" style="display: flex; gap: 1rem; justify-content: center; flex-wrap: wrap;">
{{< hextra/hero-button text="Read the Blog" link="blog" >}}
{{< hextra/hero-button text="About Me" link="about" style="secondary" >}}
</div>

<div class="hx:mt-12"></div>

## Featured Posts

<div class="hx:mt-6" style="display: grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); gap: 1.5rem; max-width: 1200px; margin: 0 auto;">

{{< hextra/feature-card
  title="Getting Started with Red Hat Developer Hub"
  subtitle="Deploy and configure the Backstage-based internal developer portal on OpenShift with GitHub, ArgoCD, and OCM integrations."
  link="blog/developer-hub-getting-started-part-1"
>}}

{{< hextra/feature-card
  title="GitOps with Tekton and ArgoCD"
  subtitle="Build a complete CI/CD pipeline using Tekton for builds and ArgoCD for deployments with GitHub webhook triggers."
  link="blog/tekton-argocd-gitops"
>}}

{{< hextra/feature-card
  title="ArgoCD with KSOPS and Age Encryption"
  subtitle="Securely manage encrypted secrets in GitOps workflows using SOPS, Kustomize, and Age encryption."
  link="blog/argocd-ksops-age"
>}}

</div>

<div class="hx:mt-16"></div>

## Topics I Write About

{{< hextra/feature-grid >}}

  {{< hextra/feature-card
    title="Platform Engineering"
    subtitle="Building developer platforms, internal tooling, and self-service infrastructure that scales with your organization."
    icon="server"
  >}}
  {{< hextra/feature-card
    title="GitOps & Automation"
    subtitle="Production-tested patterns with ArgoCD, FluxCD, Terraform, and Ansible for declarative infrastructure management."
    icon="refresh"
  >}}
  {{< hextra/feature-card
    title="Kubernetes & Containers"
    subtitle="Deep dives into container orchestration, from development clusters to production-grade deployments on OpenShift and K8s."
    icon="cube"
  >}}
  {{< hextra/feature-card
    title="Observability"
    subtitle="Implementing monitoring, logging, and tracing with Prometheus, Grafana, Loki, and modern observability stacks."
    icon="chart-bar"
  >}}
  {{< hextra/feature-card
    title="CI/CD Pipelines"
    subtitle="Crafting efficient build and deployment pipelines with Tekton, GitHub Actions, and cloud-native tooling."
    icon="lightning-bolt"
  >}}
  {{< hextra/feature-card
    title="Home Lab Chronicles"
    subtitle="Real-world experiments from my personal infrastructure—where I break things so you don't have to."
    icon="home"
  >}}
{{< /hextra/feature-grid >}}

<div class="hx:mt-12" style="text-align: center;">
  <a href="/blog/index.xml" style="display: inline-flex; align-items: center; gap: 0.5rem; color: var(--tw-prose-links); text-decoration: none;">
    {{< icon name="rss" >}} Subscribe via RSS
  </a>
</div>
