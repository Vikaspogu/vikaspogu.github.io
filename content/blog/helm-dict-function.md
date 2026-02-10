---
title: "Helm Alternative for Multiple If-Else Conditions"
date: 2021-07-08
description: "Use Helm's dict function to replace verbose if-else blocks with cleaner, more maintainable template code."
tags: ["Helm", "Kubernetes", "Templates"]
---

Recently I was in a situation where I ended up writing multiple if-else statements in the helm template. As a result, the code block wasn't clean, and I began to explore alternative ways to achieve the same behavior concisely.

[Helm dict function](https://github.com/Masterminds/sprig/blob/master/docs/dicts.md#dictionaries-and-dict-functions) perfectly fits my use case.

Before

```yaml
{{- if eq .Values.imageType "java" }}
    name: openjdk:latest
{{- else if eq .Values.imageType "nodejs" }}
    name: nodejs:latest
{{- else if eq .Values.imageType "golang" }}
    name: golang:latest
{{- else if eq .Values.imageType "python" }}
    name: python:latest
{{- end }}
```

After

```yaml
{{- $imageTypes := dict }}
{{- $_ := set $imageTypes "java" "openjdk:latest" }}
{{- $_ := set $imageTypes "nodejs" "nodejs:latest" }}
{{- $_ := set $imageTypes "python" "openjdk:latest" }}
{{- $_ := set $imageTypes "java" "openjdk:latest" }}
...
    name: {{ get $imageTypes .Values.imageType }}
```
