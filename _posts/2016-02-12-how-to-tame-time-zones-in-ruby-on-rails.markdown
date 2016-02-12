---
layout: post
title:  "How to Tame Time Zones in Ruby on Rails"
date:   2016-02-12 09:00:00
author: kjcpaas
comments: true
image: /assets/article_images/2016-02-04-how-to-tame-time-zones-in-ruby-on-rails/timezones.jpg
---

Quipper is a global company. Our developer teams are located in 3 different cities around the world: London, Tokyo, Manila. When it is 9AM in London, it is 5PM in Manila, and 6PM in Tokyo. In addition, we also service Mexico and Indonesia. All in all, there are at least 5 time zones we need to consider in development.

Time zones exist because of the Earth's rotation. In some parts of the world, it is morning, while it is evening in others. Despite how natural time zones are, many developers struggle with writing code that deals with the time differences due to time zones. It's much more tricky once [daylight saving time](https://en.wikipedia.org/wiki/Daylight_saving_time), which is implemented in some countries, is taken into consideration. To make matters more complex, it could be _another day_ (or even _another year_!) in a different time zone, i.e. Thursday night in London is Friday morning in Manila.

However, there is no need to worry! Working with timezone is not as difficult as it initially looks. Below is a list of some techniques that can be used in Ruby on Rails.

## How to set time zone

Before we proceed, how Ruby on Rails determines time zones must be known first.

There are 2 ways of doing this.

### In config/application.rb

{% highlight ruby %}
module MyApp
  class Application < Rails::Application
    config.time_zone = 'Central Time (US & Canada)'

    # ...
  end
end
{% endhighlight %}

The default setting is `UTC` (and I personally like keeping it as is, as you can see later). You can get the list of complete timezones by running `rake time:zones:all`.

### Setting Time.zone

In contrast to setting time zone in `config/application.rb`, setting Time.zone can be done anywhere within the application. When taking into account multiple time zones, be more careful about setting this in many places in the application as it is easy to lose track of the current time zone.

## Times in database must be in UTC

This is important since time zones are expressed using positive or negative offsets to UTC (e.g. UTC+8:00, UTC-5:00). In addition, Ruby has a built-in method to convert time to UTC ([Time#utc](http://ruby-doc.org/core-2.2.0/Time.html#method-i-utc)).

The main reason why I prefer keeping the default timezone as UTC is because it is much easier to keep the time zone consistent with the UTC dates and time in the database.

Also, when database times are in UTC, there is no need to consider DST switching when converting times to another time zone. When using a time zone that implements DST, there is a time (usually each year) when clocks are turned *backward* 1 hour (number of hours depends on place also). Due to this, there are days when the some times occur twice. For example, in [Washington DC](http://www.timeanddate.com/time/change/usa/washington-dc), 1:00 AM - 1:59 AM will happen twice when DST ends. It is hard to map this time correctly to other timezones. However, if the database time is in UTC, this is not problem since DST is not implemented for UTC.

## Use in\_time\_zone to convert time to another time zone

Instead of changing `Time.zone` to convert a time in another time zone, just use `in_time_zone`.

{% highlight ruby %}
Time.now
# => 2016-02-11 17:51:31 +0800

Time.now.in_time_zone('EST')
# => Thu, 11 Feb 2016 04:51:38 EST -05:00
{% endhighlight %}

## Use time zone-sensitive methods

Prefer the use of `Time.zone.now` to `Time.now`. Similarly, prefer the use of `Date.current` to `Date.today`. `Time.zone.now` and `Date.current` return the correct values based on the time zone set in `Time.zone` in Ruby on Rails.

On the other hand, `Time.now` and `Date.today` return values based on the *machine's time zone*. As an example, since I am in Manila, `Time.now` will return Manila's local time in UTC+8. Another developer who is in Tokyo who uses `Time.now` will get Tokyo's local time (UTC+9).

Overlooking this fact can result to unstable code. In fact, we encounter this when running automated tests on [CircleCI](https://circleci.com/). There are some tests that pass or fail depending on the time of the day. Having these unstable tests could mean one or both of the following:

- The test code is not time zone-sensitive
- The code being tested itself is not time zone-sensitive (BAD!)

When this happens, try to check the test code first. If the tests still fail, then most probably the problem is with the tested code.

### Side note

To encourage the use of `Date.current`, we went as far as monkey patching `Date.today` and `Date.tomorrow` in our `spec_helper`!

{% highlight ruby %}
class Date
  def self.today
    raise 'Use Date.current instead of Date.today'
  end

  def self.tomorrow
    raise 'Use Date.current + 1.day instead of Date.tomorrow'
  end
end
{% endhighlight %}

## Take advantage of Time.use_zone

When writing time zone-sensitive code, [Time.use_zone](http://api.rubyonrails.org/classes/Time.html#method-c-use_zone) is your best friend. This method overrides the setting in `Time.zone` when doing time-based calculations inside the block.

For example:

{% highlight ruby %}
Time.zone = 'UTC'
Time.zone.now
# => Thu, 04 Feb 2016 10:12:21 UTC +00:00

Time.use_zone('Asia/Tokyo') { Time.zone.now }
# => Thu, 04 Feb 2016 19:12:43 JST +09:00
{% endhighlight %}

When the code inside the block is time zone-sensitive (correctly uses `Time.zone.now` and `Date.current`), accurate time zone switching can be easily achieved within an application!
