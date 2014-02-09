---
layout: post
title: "Multi-project code coverage using Gradle and Jacoco"
date: 2014-02-09 18:38:31 +0100
comments: true
categories: [Gradle, Jacoco, code coverage, subprojects, multi-projects]
---
After playing around with a couple of different Cobertura plugins for Gradle, I realized that using the built-in Jacoco plugin is much easier to set up in a project consisting of multiple subprojects. Here is a quick solution for generating a holistic code coverage report in Gradle.

Summary:

* The root-level project doesn't contain any code, it just controls the overall build process
* The Jacoco plugin is applied to all subprojects - the jacoco task will instrument the classes and capture the execution data in `<path_to_subproject>/build/jacoco/*.exec`
* The root-level `build.gradle` uses the `JacocoReport` task to generate the holistic coverage report

``` groovy build.gradle (root-level project)

allprojects {
  apply plugin: 'jacoco'
}

task codeCoverageReport(type: JacocoReport) {

    // Gather execution data from all subprojects
    // (change this if you e.g. want to calculate unit test/integration test coverage separately)
    executionData fileTree(project.rootDir.absolutePath).include("**/build/jacoco/*.exec")

    // Add all relevant sourcesets from the subprojects 
    subprojects.each {
       sourceSets it.sourceSets.main
    }

    reports {
      xml.enabled true
      html.enabled true
      html.destination "${buildDir}/reports/jacoco"
      csv.enabled false
    }
}

// always run the tests before generating the report
codeCoverageReport.dependsOn {
    subprojects*.test
}
```


