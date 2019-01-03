---
layout: post
title:  "What we learned from Android Pie migration"
date:   2019-01-03 10:00:00
author: bopbi
comments: true
---

Google Play Store now required submitted app to use Oreo (API Level 26) as a target SDK, and Android Pie (API Level 28) also has been launched, 

We think why not set the Quipper app to target the latest SDK and AndroidX, this can be a good learning topic for our Engineer

So what we learn from the migration process

# Dependencies
Update the Android Target SDK to 28 (Pie). 

# Oreo Migration

## Android Oreo Vibration problem
Quipper have a feature to display the video download progress on the notification, unfortunately the behaviour is changed on Oreo.
Any Update on the notification progress triggers the notification tone. To handle it we need to [remove the Vibration pattern](https://stackoverflow.com/questions/46402510/notification-vibrate-issue-for-android-8-0/47646166).

## Android Service Problem
On Oreo, ```startService()``` will throw ```IllegalStateException```. This can be fixed by changing it to ```startForegroundService()```. But for other background service (like for perform a scheduled background service to delete expired video) we decided to use the [WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager/) even though it is still on alpha phase.

# Pie Migration

## Https
On Pie it is required to use HTTPS for network communication [refer to Pie Behaviour Changes](https://android-developers.googleblog.com/2018/08/introducing-android-9-pie.html), so any URL that still use HTTP need to be changed to use HTTPS, Luckily this is only affecting the video URLs and we fixed it by using a String map.

# AndroidX
We migrate the Android-support to the AndroidX Library for a better future compatibility, since the android-support will not receive any update after version 28.

We use the Migrate to AndroidX tools within the Android Studio, this is causing a quite massive changes. We also realized that some of our lib did not using the androidX, for example: autodispose, support-preferencefragment, etc. For that, we perform updates and manual changes to remove any dependencies to use the android-support library. 

After performing migration, the Unit Test is failing since the Robolectric that we use is not compatible with the androidX. For that we updated the Robolectric to version 4.1. But it turns out that the update process still need some changes to the existing Unit Test since the API to perform Activity mock is changed. To fix the failed mocking, I used some lib that have similar performance with the default Robolectric Support Library Controller, [see this gist](https://gist.github.com/bopbi/0d9b3c41e8241a5f15579c5f072a5ece) for the detail.

