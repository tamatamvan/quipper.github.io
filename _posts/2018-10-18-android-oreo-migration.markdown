---
layout: post
title:  "What we learned from Android Oreo migration"
date:   2018-10-18 10:00:00
author: bopbi
comments: true
---


# Dependencies
this is simple just update the Android Target SDK to 26 (Oreo) 

# Https
On Oreo it is required to use HTTPS for network communication, so any url that still use http need to be changed to use https, luckily this is only affecting the video url and changing it by using a String map 

# Android Oreo vibration problem
Quipper have a feature to display the video download progress on the notification, unfortunately the behaviour is changed on Oreo.
Update on the notification caused the notification tone is triggered, and to handle it we need to [remove the Vibration pattern](https://stackoverflow.com/questions/46402510/notification-vibrate-issue-for-android-8-0/47646166)

# Android Service Problem
on Oreo ```startService()``` will throw ```IllegalStateException``` this can be replaced by using ```startForegroundService()``` but for other background service (like for perform a scheduled background service for delete expired video) we decided to use the [WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager/) although it is still on alpha phase