---
layout: post
title:  "Building Better Marionette Views"
date:   2015-05-18 09:00:00
author: seth
comments: true
image: /assets/article_images/2015-05-18-better-marionette-views/little-giant-girl.jpg
---

Last year we switched from pure [BackboneJS](http://backbonejs.org/) to [MarionetteJS](http://marionettejs.com/).
For those who don't know, Marionette is a fantastic little MVC framework that builds upon the wireframe that
Backbone gives you. It does a great job of "filling in the blanks" for you so that you can concentrate on
getting your app up and running while not losing track of everything as your project grows in size.

Here I'll be sharing a few tips I learnt along the way on how you can make those Marionette views
better and easier to manage.

I'll be sharing code in [CoffeeScript](http://coffeescript.org/) as it is the language we have adopted in Quipper,
and it'll make for shorter, more concise examples.

So let's start with a typical controller and view as we may have once written it.

{% highlight coffeescript %}
class Controller extends Marionette.Controller
  show: ->
    @model = new Person
    @view = new PersonView model: @model
    app.mainRegion.show(@view) # assuming that this is our region to render views in

class PersonView extends Marionette.ItemView
  template: Tpl['person']

  ui:
    save: 'a.btnSave'
    cancel: 'a.btnCancel'

  events:
    'click @ui.save': 'clickSave'
    'click @ui.cancel': 'clickCancel'

  clickSave: (e) ->
    e.preventDefault()
    e.stopPropagation()
    @ui.save.addClass('disabled')
    @model.save().done(@saved).fail(@failed)

  saved: =>
    # respond to save

  failed: =>
    # respond to failure

  clickCancel: (e) ->
    e.preventDefault()
    e.stopPropagation()
    # navigate to somewhere else
{% endhighlight %}

### Hook yourself up with some Triggers.

Honestly, [triggers](http://marionettejs.com/docs/v2.4.1/marionette.view.html#viewtriggers) are a godsend.
They automatically prevent the default action for DOM events and turn the event into a call to
`triggerMethod` so that we can listen to the event elsewhere.

In the world of MVC this is great, because it means that the controller can do its job properly:
to control a view and respond to actions, just as a Rails app might do.

Here's how it looks with triggers to replace the events.

{% highlight coffeescript %}
class Controller extends Marionette.Controller
  show: ->
    @model = new Person
    @view = new PersonView model: @model
    @listenTo @view, 'click:save', @clickSave
    @listenTo @view, 'click:cancel', @clickCancel
    app.mainRegion.show(@view) # assuming that this is our region to render views in

  clickSave: (args) ->
    args.view.ui.save.addClass('disabled')
    args.model.save().done(@saved).fail(@failed)

  saved: =>
    # respond to save

  failed: =>
    # respond to failure

  clickCancel: (e) ->
    # navigate to somewhere else

class PersonView extends Marionette.ItemView
  template: Tpl['person']

  ui:
    save: 'a.btnSave'
    cancel: 'a.btnCancel'

  triggers:
    'click @ui.save': 'click:save'
    'click @ui.cancel': 'click:cancel'
{% endhighlight %}

So it looks like we've just moved some of the functions into the controller, and got rid of `preventDefault`
and `stopPropagation` (because Marionette does that automatically for triggers). But now we've also got
some DOM manipulation in the controller.

{% highlight coffeescript %}
args.view.$('.btn').addClass('disabled')
{% endhighlight %}

That doesn't look right. The controller shouldn't be manipulating the DOM and making UI changes, because:

- It's lazy.
- It's not very MVC.
- As our app grows, it'll become harder and harder to find which parts of the code are updating the UI.

### Marionette's triggerMethod is MAGIC. Sort of.

Whenever Marionette fires an event via `triggerMethod` which is what the `triggers` hash uses,
it automatically checks if an `on` method exists for the trigger name, where each word is separated
by colons. `do:something:cool` becomes `onDoSomethingCool`. Let's make use of that in the view.

{% highlight coffeescript %}
class PersonView extends Marionette.ItemView
  template: Tpl['person']

  ui:
    save: 'a.btnSave'
    cancel: 'a.btnCancel'

  triggers:
    'click @ui.save': 'click:save'
    'click @ui.cancel': 'click:cancel'

  onClickSave: ->
    @ui.save.addClass('disabled')
{% endhighlight %}

Thanks, triggers! So what's next?

### Pass the ball.

![To you! To me! To you! To me!](/assets/article_images/2015-05-18-better-marionette-views/rugby-pass.jpg)

Sometimes we still need to pass control back to the view from the controller. For example, we have
a handler for when the model's `save` method fails. But what if it needs to make some UI changes,
or focus on a textbox? We shouldn't call the UI from the controller.

One of the main jobs of a model is to be a mediator between View and Controller. Backbone sends
events when models are saved or have problems, so we can use this to our advantage. Let's use
`modelEvents` and stop listening to our `$.Deferred` response.

{% highlight coffeescript %}
class Controller extends Marionette.Controller
  show: ->
    @model = new Person
    @view = new PersonView model: @model
    @listenTo @view, 'click:save', @saveModel
    @listenTo @view, 'click:cancel', @cancel
    @listenTo @model, 'sync', @synced
    @listenTo @model, 'error', @failed
    app.mainRegion.show(@view) # assuming that this is our region to render views in

  saveModel: (args) ->
    args.model.save()

  synced: ->
    # respond to model being saved

  cancel: (e) ->
    # respond to cancel action

  failed: (e) ->
    # respond to model failing

class PersonView extends Marionette.ItemView
  template: Tpl['person']

  ui:
    save: 'a.btnSave'
    cancel: 'a.btnCancel'

  triggers:
    'click @ui.save': 'click:save'
    'click @ui.cancel': 'click:cancel'

  modelEvents:
    sync: 'synced'
    error: 'failed'

  onClickSave: ->
    @ui.save.addClass('disabled')

  synced: ->
    # update UI as necessary

  failed: ->
    # update UI as necessary
{% endhighlight %}

It looks longer, but that's because we have now completely separated view from controller
logic. Most of the implementation here is optional.

And check out some of the benefits!

- We have a clear workflow: The view takes care of UI and the controller takes care of action.
- We're less likely to get lost in the future when trying to debug the UI or logic.
- We have less unit tests to write because most of the triggering is done internally by Marionette.
- Our controller and view are very decoupled and can be tested independently of each other.