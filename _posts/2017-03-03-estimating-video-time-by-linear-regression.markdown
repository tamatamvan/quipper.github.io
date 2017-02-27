---
layout: post
title:  "Estimating Video Time by Linear Regression"
date:   2017-03-03 10:00:00
author: kjcpaas
comments: true
---

In Quipper, we use [Brightcove](https://www.brightcove.com/) to host our video contents.

![Quipper Video Sample](https://cloud.githubusercontent.com/assets/3772828/23352748/eb3774a2-fd03-11e6-9fbc-5b4049d986d5.png)

In most cases, the only time we need to show the video time is when the user watches the video. For this, showing the video time in Brightcove's video player itself is enough.

We have a recent update in which we have to show the video time outside of the video player. In this case, we have to get the video time from Brightcove's API. This is when we discovered that the time shown in Brightcove player, and the time from Brightcove API does not match.

![Time mismatch](https://cloud.githubusercontent.com/assets/3772828/22539207/61f06e4c-e953-11e6-98ba-7756d09f2a93.png)

In the image above, the video player shows `18:35`, while the navigation button shows `18:32` (this comes from the API). The time in video player is off by **3 seconds**! This can confuse the users so we have to do something for the video times to match, or at least be off by just 1 second.

## Why is there difference in the video time?

This is because we are actually dealing with 2 video files instead of 1. These videos are the following:

- Source video

This is the one we uploaded to Brightcove. Its video time is the one that can be obtained from Brightcove API.

- Converted Video

Once the video is uploaded, it gets converted by Brightcove to be compatible with their player. Due to the conversion process, the video time cannot be exactly the same as the source videos. The time of this converted video is the one shown in the video player.

## Observations

We observed other videos and found some patterns:

1. Converted video time is **longer** than source video time.
2. The time difference is **proportional** to the video length. This means that the time difference is smaller when the video is short, and gets bigger as the video length gets longer.

This relationship can be visualized by a graph:

![Graph](https://cloud.githubusercontent.com/assets/3772828/23353842/7946a3da-fd09-11e6-98d4-d2c2c46cd877.png)

The actual relationship might not be linear at all but for the usual time range of our videos (4 minutes to 40 minutes), assuming a linear relationship is enough.

## Solution

The approach was to find the line that would best fit the relationship between the source video time and the converted video time. We got some sample videos, plotted their times in a table.

From this, we got the best fit line with the equation `y = 1.002x + 0.4253`, where `y` is **converted video time**, and `x` is **source video time**.

![Best fit line](https://cloud.githubusercontent.com/assets/3772828/22541505/a2a02af0-e961-11e6-9a7e-1cdfb17f37af.png)

We adjusted the `y-intercept` to `0.7` from `0.4253` to take into account that our API applies `floor` function to store the the time in seconds (hence the actual source time is longer by less than 1 second).

We applied the final formula `y = 1.002x + 0.7` to estimate the converted video time shown in our UI elements. The result is in the table below.

![Table](https://cloud.githubusercontent.com/assets/3772828/22541488/7b634d0a-e961-11e6-91d0-109801c0ce38.png)


As shown in the table, there are still some cases in which the estimated time is not the same as the time shown by the player. Since this is an estimate, it's impossible to be exact. However, we managed to achieve our goal of getting within 1 second of the actual player video time. :tada:

## Conclusion

From solving this problem, we can learn the following things:

- Video conversion does not only affect the video quality or size, but also the time.
- Part of a good UI is consistency in the displayed information. No matter how good-looking the UI is, if the information displayed is inconsistent, it can cause a lot of confusion.
- Even though web developers don't usually use Math in their line of work, they should be prepared to apply the concept to solve the problems encountered.
