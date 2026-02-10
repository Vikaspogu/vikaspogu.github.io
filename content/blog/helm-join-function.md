---
title: "Helm Join Strings in a Named Template"
date: 2020-10-08
description: "Use Helm's join function in named templates to dynamically construct ServiceAccount names and other string values."
tags: ["Helm", "Kubernetes", "Templates"]
slug: "helm-join-strings"
---

The key benefit of Helm is that it helps reduce the configuration a user needs to provide to deploy applications to Kubernetes. With Helm, we can have a single chart that can deploy all the microservices.

## Unique ServiceAccount

Recently we wanted to create a unique service account for each microservice, so all microservices don't share the same service account using the helm template. In our example, we will append the release name with the service account name.

### Using join function

```yaml
{{ (list .Values.serviceAccount.name (include "openjdk.fullname" .) | join "-") }} 
```

### Adding fail condition

```yaml
{{- if .Values.serviceAccount.name -}} 
  {{ (list .Values.serviceAccount.name (include "openjdk.fullname" .) | join "-") }} 
{{- else -}} 
  {{- fail "Please enter a name for service account to create." }} 
{{- end -}}
```

Full example

```yaml
{{- define "openjdk.serviceAccountName" -}}
{{- if .Values.serviceAccount.create -}} 
  {{- if .Values.serviceAccount.name -}} 
    {{ (list .Values.serviceAccount.name (include "openjdk.fullname" .) | join "-") }} 
  {{- else -}} 
    {{- fail "Please enter a name for service account to create." }} 
  {{- end -}}
{{- else -}} 
  {{- if .Values.serviceAccount.name -}} 
    {{ (list .Values.serviceAccount.name (include "openjdk.fullname" .) | join "-") }} 
  {{- else -}} 
    "default" 
  {{- end -}}
{{- end -}}
{{- end -}}
```

Another way to manipulate strings in `helpers.tpl` is using `printf`

```yaml
{{- define "ocp-openjdk.hostname" -}}
{{- printf "%s-%s%s" .Release.Name .Release.Namespace (.Values.subdomain | default ".apps.amp01.nonprod" ) -}}
{{- end -}}
```

## Resources

* [Sprig functions](https://github.com/Masterminds/sprig)
