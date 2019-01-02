---
layout: post
title:  "What we learned from Android Oreo migration"
date:   2018-10-18 10:00:00
author: bopbi
comments: true
---


# Dependencies
this is simple, just update the Android Target SDK to 26 (Oreo). 

# Https
On Oreo it is required to use HTTPS for network communication [refer to Oreo Behaviour Changes](https://developer.android.com/about/versions/oreo/android-8.0-changes), so any URL that still use HTTP need to be changed to use HTTPS, Luckily this is only affecting the video URLs and we fixed it by using a String map.

# Android Oreo Vibration problem
Quipper have a feature to display the video download progress on the notification, unfortunately the behaviour is changed on Oreo.
Any Update on the notification progress triggers the notification tone. To handle it we need to [remove the Vibration pattern](https://stackoverflow.com/questions/46402510/notification-vibrate-issue-for-android-8-0/47646166).

# Android Service Problem
On Oreo, ```startService()``` will throw ```IllegalStateException```. This can be fixed by changing it to ```startForegroundService()```. But for other background service (like for perform a scheduled background service to delete expired video) we decided to use the [WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager/) even though it is still on alpha phase.