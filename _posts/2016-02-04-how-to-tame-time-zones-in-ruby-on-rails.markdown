---
layout: post
title:  "How to Tame Time Zones in Ruby on Rails"
date:   2016-02-04 09:00:00
author: kjcpaas
comments: true
image: /assets/article_images/2016-02-04-how-to-tame-time-zones-in-ruby-on-rails/timezones.jpg
---

Quipper is a global company. Our developer teams are located in 3 different cities around the world: London, Tokyo, Manila. When it is 9AM in London, it is 5PM in Manila, and 6PM in Tokyo. In addition, we also service Mexico and Indonesia. All in all, there are at least 5 time zones we need to consider in development.

Time zones exist because of the Earth's rotation. In some parts of the world, it is morning, while it is evening in others. Despite how natural time zones are, many developers struggle with writing code that deals with the time differences due to time zones. It's much more tricky once [daylight saving time](https://en.wikipedia.org/wiki/Daylight_saving_time), which is implemented in some countries, is taken into consideration.

However, there is no need to worry! Working with timezone is not as difficult as it initially looks. Below is a list of some techniques that can be used in Ruby on Rails.

## Times in database must be in UTC

This is important since time zones are expressed using positive or negative offsets to UTC (e.g. UTC+8:00, UTC-5:00). In addition, Ruby has a built-in method to convert time to UTC ([Time#utc](http://ruby-doc.org/core-2.2.0/Time.html#method-i-utc)).

## Use time zone-sensitive methods

Prefer the use of `Time.zone.now` to `Time.now`. Similarly, prefer the use of `Date.current` to `Date.today`. `Time.zone.now` and `Date.current` return the correct values based on the time zone set in `Time.zone` in Ruby on Rails.

On the other hand, `Time.now` and `Date.today` returns values based on the *machine's time zone*. As an example, since I am in Manila, `Time.now` will return Manila's local time in in UTC+8. Another developer who is in Tokyo who uses `Time.now`will get Tokyo's local time.

Overlooking this fact can result to unstable code. In fact, we encounter this when running automated tests on [CircleCI](https://circleci.com/). There are some tests that pass or fail depending on the time of the day. Having this unstable tests could mean one or both of the following:

- The test code is not time zone-sensitive
- The code being tested itself is not time zone-sensitive (BAD!)

When this happens, try to check the test code first. If the tests still fail, then most probably the problem is with the tested code.

## Take advantage of Time.use_zone

When writing time zone-sensitive code, [Time.use_zone](http://api.rubyonrails.org/classes/Time.html#method-c-use_zone) is one of your best friends. This method overrides the setting in `Time.zone` when doing time-based calculations inside the block.

For example:

{% highlight ruby %}
Time.zone = 'UTC'
Time.zone.now
# => Thu, 04 Feb 2016 10:12:21 UTC +00:00

Time.use_zone('Asia/Tokyo') { Time.zone.now }
# => Thu, 04 Feb 2016 19:12:43 JST +09:00
{% endhighlight %}

When the code inside the block is time zone-sensitive (correctly uses `Time.zone.now` and `Date.current`), accurate time zone switching can be easily achieved within an application!
