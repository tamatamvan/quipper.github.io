---
layout: post
title:  "When AJAX turns up late for the party"
date:   2015-07-22 09:30:00
author: seth
comments: true
image: /assets/article_images/2015-07-22-ajax-late-party/empty-party.jpg
---

Imagine that you've been invited to your best mate Derek's birthday bash at his flat.
Owing to traffic and bus strikes you arrive later than usual to the party, and
when you knock... nobody answers!

Apparently Derek decided that the local Nando's was more banterous than his flat, or perhaps
he's such a hero that too many people turned up, so they all moved to a bigger venue at the
last minute. But he didn't text you the new place, he didn't even leave a note on the door
to say "Sorry we missed you, bud!"

Maybe Derek's just not a very thoughtful friend after all.

## We can all be a bit like Derek.

Sadly this happens a lot in single-page apps. I've seen plenty of code that looks
something like the following:

```coffeescript
class MainView extends Marionette.ItemView
  onRender: ->
    @model.fetch().done =>
      contentView = new ContentView(@model)
      @contentRegion.show(contentView)
```

It seems harmless. But what if AJAX is late to the party? What if the model takes
ages to fetch and the user decides to go to another part of the site? Eventually
the model will either complete or fail, but by this time the app may have long since
moved on from the old view.

When this happens our console can start to look like this.

![Embarrassing console errors](/assets/article_images/2015-07-22-ajax-late-party/undefined-errors.png)

Most users may not bother to look in their console. But for those who do, this
is pretty unprofessional. Plus if we're logging all these errors, we'll be clogging
up the logs with minor annoyances.


## At least leave a note.

It's really not too hard to fix this kind of problem, it just requires making a habit
of always remembering:

> **If your process is going to come back asynchronously, never
assume that your view still exists, or the page state is what it was before.**

The easiest measure is a quick and silent exit. This will prevent most JavaScript errors.

```coffeescript
class MainView extends Marionette.ItemView
  onRender: ->
    @model.fetch().done =>
      return if @isDestroyed
      contentView = new ContentView(@model)
      @contentRegion.show(contentView)
```

Perhaps a cleaner measure is to stop using `$.Deferred()` callbacks altogether, since a model
will send `sync` and `error` events after fetching.

```coffeescript
class MainView extends Marionette.ItemView
  modelEvents:
    sync: 'createContentView'

  onRender: ->
    @model.fetch()

  createContentView: ->
    contentView = new ContentView(@model)
    @contentRegion.show(contentView)
```


## Even better, tell your guests to cancel.

A good host ought to let his guests know if the evening's plans have changed. And a good
single-page app is no different.

XMLHttpRequests allow us to abort them. If we want to be very kind to our late arrivals,
and thus our users, we could tell them to just give up and not bother any more.
A very simple implementation is to track our requests and use the `abort()` method to cancel them.

```coffeescript
class MainView extends Marionette.ItemView
  modelEvents:
    sync: 'createContentView'

  initialize: ->
    @requests = []

  onRender: ->
    @requests.push(@model.fetch())

  onDestroy: ->
    req.abort() for req in @requests

  createContentView: ->
    contentView = new ContentView(@model)
    @contentRegion.show(contentView)
```

These are just a couple of tips to being a more considerate host. Above all, always remember
that network calls can take any length of time, even living longer than the 
object that spawned them.
