---
layout: post
title:  "Kotlin Android migration"
date:   2018-10-07 10:00:00
author: bopbi
comments: true
---

Recently Quipper Android migrated from java to Kotlin, let me shared some of the issue that we got during migration process:

## Kotlin gradle configuration/dependencies
For the first step to migration we follow the [Kotlin Gradle guideline](https://kotlinlang.org/docs/reference/using-gradle.html)

- add mavenCentral and kotlin dependecies url
- add the kotlin plugin and set the kotlin version
- replace any ```annotationProcessor``` to ```kapt```
- implement all of the changes above to all module, since the existing source is consist of multiple modules

## Existing library problem
The first problem we encountered is the conflict with the [Lombok](https://projectlombok.org/) library, during compilation it always shows error for any property annotated with ```@Getter```, for this we need to manually remove the annotation and generate the required getter and setter, but still it affected a lot of source code, for this process it took almost 2-3 days (change and review) and during migration we still maintain any unused getter and setter

## Existing java source code
[Kotlin visibility](https://kotlinlang.org/docs/reference/visibility-modifiers.html) access is different with java, on the existing code some implementation class is designed to be injected, so we have to violate/detour from the existing architecture.

Java Nullability, the existing code is using a lot of ```Callback``` mechanism, and since java param/property is nullable by default, on the Kotlin code any property/function that are comming from existing code is nullable unless it proven otherwise.
