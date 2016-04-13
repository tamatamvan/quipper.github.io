---
layout: post
title:  "Fix your Marionette/Backbone memory leaks."
date:   2016-04-08 10:00:00
author: seth
comments: true
image: /assets/article_images/2016-04-08-fix-your-marionette-memory-leaks/leeks.jpg
---

Recently I noticed some strange behaviour while browsing through the our single-page app. An event handler was being called multiple times, which increased the more that I used the site. Obviously the handler wasn't being released.

You can imagine the event handler as being something as simple as the following killer app:

```coffeescript
class MyModel extends Backbone.Model
  initialize: ->
    # This model wants to listen to some global event emitted by the app
    # and perform an action that is useful for the view that uses it.
    App.reqres.on('flash', @someEvent)

  someEvent: ->
    alert('Aaahhh!')

view = new MyView(model: new MyModel)
KillerApp.mainRegion.show(view)
```

This seems to work fine. The view is shown with the model, and when the app sends our message `flash`, an alert message pops up. Now let's assume we're finished with our view and its model.

```coffeescript
anotherView = new AnotherView()
KillerApp.mainRegion.show(anotherView)
```

At this point, Marionette has destroyed our previous view and released its resources.. right? However, try triggering `flash` again, and you'll find that the alert appears! Our model has not been released properly.

## I want to break free.

I was mistaken in believing that when a view is destroyed and its listeners released, the model's listeners are also released. They are not! We **must** tell our model manually to **stop listening**.

```coffeescript
class MyView extends Marionette.ItemView
  onDestroy: ->
    # cleanup
    @model.stopListening()
```

This can become even more complicated if you have a collection whose children are all listening to events on other objects. You must remember to not just unbind the collection but also all its children.

```coffeescript
class MyCollection extends Backbone.Collection
  model: MyModel

  stopListening: ->
    model.stopListening() for model in @models
    super

class MyCollectionView extends Marionette.CollectionView
  onDestroy: ->
    @collection.stopListening()
```

## Who wants to live forever?

This all comes down to JavaScript's garbage collection. Normally we forget about it because it's pretty intelligent. If an object is no longer needed and its references have all been cleared out, then JavaScript will clean it up for us.

But if that object is **listening to events on a persistent object**, then it will want to live forever as long as it is listening! Even worse, the immortality of this object then passes on to anything else listening to it, and so on. In most occasions though, we don't want models or collections to live past the lifetime of the view.

So don't forget to tell your models, controllers, and other objects, to stop listening after the view has been destroyed.

![](/assets/article_images/2016-04-08-fix-your-marionette-memory-leaks/you-may-go.png)
