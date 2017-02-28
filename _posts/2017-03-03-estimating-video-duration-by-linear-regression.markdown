---
layout: post
title:  "Estimating Video Duration by Linear Regression"
date:   2017-03-03 10:00:00
author: kjcpaas
comments: true
---

In Quipper, we use [Brightcove](https://www.brightcove.com/) to host our video contents.

![Quipper Video Sample](https://cloud.githubusercontent.com/assets/3772828/23352748/eb3774a2-fd03-11e6-9fbc-5b4049d986d5.png)

In most cases, the only time we need to show the video duration is when the user watches the video. For this, showing the video duration in Brightcove's video player itself is enough.

We have a recent update in which we have to show the video duration outside of the video player. In this case, we have to get the video duration from Brightcove's API. This is when we discovered that the duration shown in Brightcove player and the duration from Brightcove API do not match.

![Time mismatch](https://cloud.githubusercontent.com/assets/3772828/22539207/61f06e4c-e953-11e6-98ba-7756d09f2a93.png)

In the image above, the video player shows `18:35`, while the navigation button shows `18:32` (this comes from the API). The duration in video player is off by **3 seconds**! This can confuse the users so we have to do something for the video durations to match, or at least be off by just 1 second.

## Why is there difference in the video duration?

This is because we are actually dealing with 2 video files instead of 1. These videos are the following:

- Source video

This is the one we uploaded to Brightcove. Its video duration is the one that can be obtained from Brightcove API.

- Converted Video

Once the video is uploaded, it gets converted by Brightcove to be compatible with their player. Due to the conversion process, the converted video duration cannot be exactly the same as the source video's. The time of this converted video is the one shown in the video player.

## Observations

We observed other videos and found some patterns:

1. Converted video duration is **longer** than source video duration.
2. The time difference is **proportional** to the video duration. This means that the difference in duration is smaller when the video is short, and gets bigger as the video gets longer.

This relationship can be visualized by a graph:

![Graph](https://cloud.githubusercontent.com/assets/3772828/23353842/7946a3da-fd09-11e6-98d4-d2c2c46cd877.png)

The actual relationship might not be linear at all but for the usual duration range of our videos (4 minutes to 40 minutes), assuming a linear relationship is enough.

## Solution

The approach was to find the line that would best fit the relationship between the source video duration and the converted video duration. We got some sample videos are plotted their duration in a table.

From this, we got the best fit line with the equation `y = 1.002x + 0.4253`, where `y` is **converted video duration**, and `x` is **source video duration**.

![Best fit line](https://cloud.githubusercontent.com/assets/3772828/22541505/a2a02af0-e961-11e6-9a7e-1cdfb17f37af.png)

We adjusted the `y-intercept` from `0.4253` to `0.7` to take into account that our API applies `floor` function to store the duration in seconds (hence the actual source duration is longer by at most 1 second.

We applied the final formula `y = 1.002x + 0.7` to estimate the converted video duration shown in our UI elements. The result is in the table below.

![Table](https://cloud.githubusercontent.com/assets/3772828/22541488/7b634d0a-e961-11e6-91d0-109801c0ce38.png)


As shown in the table, there are still some cases in which the estimated duration is not the same as the duration shown by the player. Since this is an estimate, it's impossible to be exact. However, we managed to achieve our goal of getting within 1 second of the actual player video duration. :tada:

## Conclusion

From solving this problem, we can learn the following things:

- Video conversion does not only affect the video quality or size, but also the duration.
- Part of a good UI is consistency in the displayed information. No matter how good-looking the UI is, if the information displayed is inconsistent, it can cause a lot of confusion.
- Solutions to problems encountered in web development may not always be found in its usual scope. Web developers must always be prepared to look for answers beyond their current realm of knowledge. This is one great way to improve skills.
