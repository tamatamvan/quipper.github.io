---
layout: post
title:  "How missing translations in Rails can mess with your markup"
date:   2016-08-08 10:00:00
author: kjcpaas
comments: true
---

Last week, we experienced something that we never thought will happen. Some missing translations messed up the content and layout of one of our landing pages.

This is how we expected it to look like:

![](https://cloud.githubusercontent.com/assets/3772828/17467365/191cc510-5d50-11e6-9a3e-5136f6c3a2c7.png)

However, it looked like this:

![](https://cloud.githubusercontent.com/assets/3772828/17333416/e54cff24-5904-11e6-8a68-dd0e19a9b9b0.png)

Furthermore, in mobile, it messed the page layout by content div being pushed inwards due to the long string.

![](https://cloud.githubusercontent.com/assets/1891005/17333948/5ac59c36-58cc-11e6-9f7f-a2b5179bde08.png)

## Investigation

We found out that it's due to I18n's missing translation message.

![This image show missing translation error for `es-MX` but this also happens to other locales like `th`](https://cloud.githubusercontent.com/assets/1628558/17333775/941efe42-58cb-11e6-8f40-82413af4fb9d.png)

Notice that the message is enclosed in a `<span></span>` element. However, since the translated segment was being used as the value for the images' alt, the span element messed with the alt's value. Unfortunately, the segments being used as image alt were not yet available for certain locales.

This code was tracked to [this line](https://github.com/rails/rails/blob/c1dbb13eacf0e579f351a46c9ee2ec845ae0cc2d/actionview/lib/action_view/helpers/translation_helper.rb#L108) in `ActionView#TranslationHelpers`.

## Solution

After reading the method in `ActionView#TranslationHelpers`, we found a way on how to workaround this problem. By passing a `:default` paramater in or `t` method call in the view, we were able to override the default `span` return element on missing translation.

__Before fix__

```ruby
= image_tag 'my_image.gif', alt: t('my.missing.segment')
```

We set `:default` to a blank string.

__The fix__

```ruby
= image_tag 'my_image.gif', alt: t('my.missing.segment', default: '')
```

After applying the simple fix, the images showed as expected!

![](https://cloud.githubusercontent.com/assets/3772828/17467365/191cc510-5d50-11e6-9a3e-5136f6c3a2c7.png)
