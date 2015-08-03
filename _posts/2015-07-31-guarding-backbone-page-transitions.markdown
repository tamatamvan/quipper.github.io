---
layout:   post
title:    "Guarding Backbone Page Transitions"
date:     2015-07-31 12:00:00
author:   kjcpaas
comments: true
---

When navigating sites, users sometimes encounter the popup below when navigating away from a page.

![](http://i.gyazo.com/856f85d351defb43ecaa41119a41cca0.png)

This confirmation is important in times when data might be lost when leaving a page.

Normally, this is done by:

{% highlight coffeescript %}
window.onbeforeunload = -> 'Your new changes will be lost.'
{% endhighlight %}

However, Backbone applications are single-page applications. In this case, moving to another page in Backbone does not reload a page.

Page navigations are managed by Backbone and unfortunately, this does not trigger the `beforeunload` event. It will only trigger when closing the window/tab or pressing the "Refresh" buttons in the browser, not when pressing "Back" or navigation links on the page. Therefore, page transition guard is incomplete (and this is not good).

## Solution

To solve this problem, Veracross' [backbone-beforepopstate](https://github.com/veracross/backbone-beforepopstate) can be used. This is dependent on Backbone.js and jQuery so make sure to include them, too.

backbone-beforepopstate adds 4 events that are triggered during Backbone navigation.

- `popstate` - triggered by pressing 'Back' button on browser
- `pushstate` - triggered by navigating by interacting with page elements
- `beforepopstate` - triggered before page transition is applied on `popstate`
- `beforepushstate` - triggered before page transition is applied on `pushstate`

Since what we're trying to achieve is to guard page transition before it actually happens, `beforepushstate` and `beforepopstate` will be used.

### Activating backbone-beforepopstate

After loading Backbone and jQuery, call

{% highlight coffeescript %}
Backbone.addBeforePopState(Backbone)
{% endhighlight %}

This will override Backbone's default navigation methods to add mechanisms to trigger the above events.

### Constructing the handlers

One thing to keep in mind for handlers is that if it returns `null`, it will not guard the page transition (popup won't be shown). On the other hand, if a string is returned, this string will be shown in the popup.

![](http://i.gyazo.com/4b097d47b068b013a79ec4ceda0d6d83.png)

Handlers can be attached using the following code:

{% highlight coffeescript %}
guardPage = (e) -> 'Your new changes will be lost.'
dontGuardPage = (e) -> null

# Will guard page transition
$(window).on('beforepopstate beforepushstate', guardPage)

# Will not guard page transition (since handler returns null)
$(window).on('beforepopstate beforepushstate', dontGuardPage)
{% endhighlight %}

After this, pressing "Back" button and navigating by interacting with page elements are now guarded!

**NOTE:** Popup for `beforepopstate` and `beforepushstate` uses `window.confirm()` so instead of the "Leave this Page" and "Stay on this Page" buttons, it shows "OK" and "Cancel".

![](http://i.gyazo.com/4b097d47b068b013a79ec4ceda0d6d83.png)

**How about `onbeforeunload`?**

The same handlers can be used for `beforeunload` event

{% highlight coffeescript %}
$(window).on('beforepopstate beforepushstate beforeunload', guardPage)
{% endhighlight %}

Finally, the page is fully guarded!

## More tips

- The different events received by a single handler can be differentiated by the `type` property. Hence, if a slightly different behavior is needed for a certain event, this can be used.

- Events from backbone-beforepopstate have a `fragment` property. This is the URL fragment that the application would like to transition to. If only certain transitions should be permitted, this URL can be compared with the current URL to show the popup correctly.

{% highlight coffeescript %}
# Questionnaire path: `/question/:question_number`

guardQuestionnaireExit = (e) ->
  currentUrl = Backbone.history.fragment
  toUrl = e.fragment
  prompt = 'Your answers to the questionnaire will be lost.'
  questionnairePathRegex = /^question\//

  # Do not guard if currently not in questionnaire
  return unless questionnairePathRegex.exec(currentUrl)

  # Guard if answering questionnaire and closing/refreshing window
  return prompt if e.type == 'beforeunload'

  # Guard if going to a page other than questionnaire from questionnaire
  return prompt unless questionnairePathRegex.exec(toUrl)

$(window).on('beforepopstate beforepushstate beforeunload', guardQuestionnaireExit)
{% endhighlight %}
