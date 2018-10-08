---
layout: post
title:  "Kotlin Android migration"
date:   2018-10-07 10:00:00
author: bopbi
comments: true
---

Recently Quipper Android migrated from java to Kotlin, let me share some of the issues that we got during migration process:

## Kotlin gradle configuration/dependencies
For the first step of the migration we followed the [Kotlin Gradle guideline](https://kotlinlang.org/docs/reference/using-gradle.html)

- add mavenCentral and kotlin dependecies url
- add the kotlin plugin and set the kotlin version
- replace any ```annotationProcessor``` to ```kapt```
- implement all of the changes above to all module, since the existing source is consist of multiple modules

## Existing library problem
The first problem we encountered was a conflict with the [Lombok](https://projectlombok.org/) library during compilation, it always shows error for any property annotated with ```@Getter```. For that reason we need to manually remove the annotation and generate the required getter and setters which consequentially affected a lot of code. The process took almost 2-3 days, and during migration we still maintain the unused getter and setters.

## Existing java source code
[Kotlin visibility](https://kotlinlang.org/docs/reference/visibility-modifiers.html) access is different with java and some *implementation classses* from existing code are designed to be injected, so we have to violate/detour from the existing architecture.

Java Nullability, the existing code is using a lot of ```Callback``` mechanism, and since java param/property is nullable by default, on the Kotlin code any property/function that are comming from existing code is nullable unless it proven otherwise.
