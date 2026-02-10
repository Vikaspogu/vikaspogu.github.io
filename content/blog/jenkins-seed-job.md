---
title: "Configure Jenkins Pipeline Job with JCasC"
date: 2020-08-18
description: "Define declarative pipeline jobs from Bitbucket repositories using Jenkins Configuration as Code plugin."
tags: ["Jenkins", "JCasC", "CI/CD", "Pipeline"]
slug: "jenkins-pipeline-jcasc"
---

In one of my previous posts, I discussed configuring multibranch pipeline seed jobs using Jenkins configuration as code [plugin](https://github.com/jenkinsci/configuration-as-code-plugin)

Below is an example of configuring a declarative pipeline job from a bitbucket repo, running at midnight every day

```yaml
- script: >
    pipelineJob('sample-job') {
      definition {
          cpsScm {
            scriptPath 'job1/Jenkinsfile' ## If the Jenkins job is in a nested folder
            scm {
              git {
                  remote {
                    url 'https://github.com/Vikaspogu/sample-repo'
                    credentials 'sample-creds'
                  }
                  branch '*/master'
                  extensions {}
              }
            }
            triggers {
              cron('@midnight')
            }
          }
      }
    }
```
