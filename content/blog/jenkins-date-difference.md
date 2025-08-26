+++ 
date = 2020-07-07
title = "Jenkins pipeline date helper functions"
description = "Jenkins pipeline function to get days between dates"
slug = "" 
tags = ["Jenkins","groovy","date"]
categories = []
externalLink = ""
series = []
socialShare=true
+++

The below snippet shows how to calculate days between two dates in the Jenkins pipeline. `@NonCPS` annotation is functional when you have methods that use objects that aren't `serializable`.

```groovy
import java.text.SimpleDateFormat

stages {
    stage("Calculate days") {
      steps {
        script {
          def daysRemaining = getRemainingDays("2020-06-25")
        }
      }
    }
  }
}

@NonCPS
def getRemainingDays(previousDate){
  def currentDate = new Date()
  String currentTimeFormat= currentDate.format("yyyy-MM-dd")
  def oldDate = new SimpleDateFormat("yyyy-MM-dd").parse(previousDate)
  return currentTimeFormat-oldDate
}
```

Calculate if the date is greater than 14 days

```groovy
import java.time.LocalDate
import java.time.format.DateTimeFormatter

stages {
    stage("Check if date is older than 14 days") {
      steps {
        script {
          if (checkIfOtherThan("2020-06-25")){
            println "Date is greater than 14 days"
          }
        }
      }
    }
  }
}

@NonCPS
def checkIfOlderThan(previousDate){
  def dateFormat = DateTimeFormatter.ofPattern("yyyy-MM-dd")
  def currentDate = LocalDate.now().format(dateFormat);
  def projectpreviousDate = LocalDate.parse(previousDate,  dateFormat)
  if (LocalDate.parse(currentDate, dateFormat).minusDays(14) > projectpreviousDate) {
    return true;
  }
  return false;
}
```
