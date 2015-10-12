---
layout: post
title:  "Where is my Ruby method defined?"
date:   2015-10-12 11:00:00
author: seth
comments: true
image: /assets/article_images/2015-10-12-where-is-my-ruby-method-defined/junk.jpg
---

Recently I noticed that the `present?` method was exhibiting weird behaviour on one of our models.

{% highlight ruby %}
Chapter.new.present?
 => false # wut!?
{% endhighlight %}


## source_location

After the initial panic wears off, the sane thing to do in this situation is to find out what `present?` is actually doing.
Fortunately, Ruby 1.9+ allows us to find exactly where a method is defined: `source_location` !

{% highlight ruby %}
Chapter.new.method(:present?).source_location
 => [".../gems/activesupport-4.1.12/lib/active_support/core_ext/object/blank.rb", 16]
{% endhighlight %}

So far so good. `present?` is defined by `ActiveSupport` just like we expect it to be. But let's confirm ActiveSupport's definition.

{% highlight ruby %}
def present?
  !blank?
end

def blank?
  respond_to?(:empty?) ? !!empty? : !self
end
{% endhighlight %}

And now back to our console again...

{% highlight ruby %}
Chapter.new.method(:empty?).source_location
 => [".../app/models/chapter.rb", 25]
{% endhighlight %}

Aha! We do indeed have a bogus re-definition of ActiveSupport. A quick fix later and everything is behaving normally.

{% highlight ruby %}
Chapter.new.present?
 => true
{% endhighlight %}


## TIL

In a big app with lots of gems it can be hard to know exactly where a method is defined, especially
since it can be overwritten and redefined numerous times. Ruby, as always, makes it easy.
