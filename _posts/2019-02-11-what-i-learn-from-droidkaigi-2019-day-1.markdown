---
layout: post
title:  "What i learned from DroidKaigi 2019 - Day 1"
date:   2019-02-17 10:00:00
author: bopbi
comments: true
---


## Day 1

### 1. What will be changed on tests in multi-module projects
[Youtube](https://www.youtube.com/watch?v=BT77YDDr4hY) - [Slide](https://speakerdeck.com/tkmnzm/how-change-testing-in-modular-architecture) - [Code](https://github.com/tkmnzm/MultiModulePlayground)

On this talk the speaker showed me that one of the benefit from splitting the app into multiple module that we can test the module feature without building the whole APK thus speed up the development time and testing time.

Currently we mostly divided the app based on the layers like UseCase, Repo, etc.. it does also effecting the how we use the DI, mainly it use to provide each component on those layer (ex: provideARepository, provideAUseCase, etc..).

Before we implement the module separation we need to think what is goal/purpose of a module, and what is the best way to test the module, so the speaker showed that a module should grouped based on a feature, so the main app will be injected by Feature (the source code it showed that the ViewModel is injected with feature)

For the module testing i saw that the every module might have a dependency with other module, the tester suggest that each module able to be tested independently

New Things that i learn:
- Testing composition : Unit Test > Integration Test > UI Test
- [Mutation Testing](https://en.wikipedia.org/wiki/Mutation_testing) which is a test to validate our unit test (by changing the source code, it will show how the unit test responded to the code change), and one of the lib to perform a Mutation Testing is [pittest](http://pitest.org/) and the source code is also provide an example of pittest usage on the android project

### 2. Dependency Injection using Dagger2 in multi module projects
[Youtube](https://youtu.be/wN2ZBtqPUz4) - [Slide](https://speakerdeck.com/kgmyshin/android-multi-module-with-dagger)

* please note that on this talks the DI is using Dagger Android support (that use ```@ContributesAndroidInjector```)

On this talks the speaker showed me that on a Monolithic app, the steps to implement DI using Dagger is divided into:
1. Module Declaration
2. Fragment Module declaration
3. Component Declaration
4. Initialize the Component on the Application#onCreate
5. Use the injection

but on a multimodule app we cannot inject the app with a multiple component like:
```java
onCreate() {

  DaggerAComponent.create().inject(this)
  DaggerBComponent.create().inject(this)
}
```
, it will crash the app since the application can only be injected by a single component

so the steps for DI implementation on a mutimodule will be changed into this:
1. Module Declaration
2. Fragment Module declaration
3. Injector Declaration (new)
4. Component Declaration (need changes)
5. Initialize the Component on the Application#onCreate (need changes)
6. Use the injection

the injector in the number 3, is a way to add injection dynamically it will extend the ```HasDispatchingInjector``` class

the speaker also shared a glimpse about how we can use the Dynamic Injection with the Android Dynamic Feature (Download part of the app when it is needed), since we cannot perform an injection on the directly on the Application#onCreate, we need to inject it by use the ```moduleFragmentInjector```

* the code in the slide is quite detail so even a non japanese will understand about what the it means

### 3. The GREATEST usecase ever! an answer to defining business logic
[Youtube](https://youtu.be/bw8bckLSKiM) - [Slide](https://speakerdeck.com/kiuchikeisuke/bokufalsekangaetazui-qiang-falseusecasefalsezuo-rifang-aruihabizinesurozitukutohananikatoiu1tufalsehui-da) - [Code](https://github.com/kiuchikeisuke/RakutenCall)

On this talks the speaker sharing his experience about the difference on Business Logic, and a Use Case when work on an App (Rakuten Call).
The Speaker think that a Business Logic is a Specification that are known by the Business Team, and the Use Case are a Business Logic that are splitted by Case, so a Use Case can be decided when discussing with the Business Team

On the provided code i learn that:
- The Business Logic and The Use Case are possible to extends the same Class
- The Business Logic is use to define the whole flow
- Use Case is set as a Constraint Logic
- The Package Groupping is different between Use Case and Business Logic although it extends the same class, so package groupping is important

### 4. The good and bad of modern app architecture
[Youtube](https://youtu.be/0gEwwXSmbww) - [Slide](https://www.slideshare.net/fast-retailing/the-good-and-bad-of-modern-app-architecture)

The speaker shared his experience when implementing a modern app architecture (Layered Components), he shared some stories about the decisions like
The Good:

The Bad:

### 5. Grid systems and Android
[Youtube](https://youtu.be/wt0_AWInokY) - [Slide](https://speakerdeck.com/soham/grid-systems-and-android-droidkaigi-2019)

For this talks i learn that:
- 4dp Grid is used for icon and typography, and the rest should use 8dp Grid
- Keyline is very important, it helped a lot as a reference Guide to speed up the development process
- Implement a Responsive design, by using the number of column and combine it with the breakpoint rule
- Tools to display Grid on the Android, so it can help to provide feedback for the Grid Implementation, like the Keyline Pusing, Material Cue, etc.

### 6. From Monolithic to Modularized codebase with Dagger
[Youtube](https://youtu.be/31d1HIw64RI) - [Slide](https://drive.google.com/open?id=1rICcpOK-5ly61KLqkyTlhgvAPKL18OPs)

Since the things that commonly injected mainly are ViewModel, UseCase/Service, and Repository. The speaker suggest to groupped those part in a SubComponent, each SubComponent may represent a Feature on the app, and those SubComponent may grouped into a Component (on the slides it called the AppComponent).

The modularization also show some challenge like:
- Since it is possible that multiple feature to access the same Data Source (API / Repository)
- Some feature may require a special android permission
- The Feature may triggered from a BrodcastReceiver, and Service

Based on the issue, the speaker suggested that the main app not calling the feature module directly, but create another module that will handle any feature module as an abstraction layer (the speaker call it a data module)

The Dynamic Module also show some challenge since it will change how we call another Feature, one of the changes are calling another Activity by its Class Name (using String combined with reflection)