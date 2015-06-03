---
layout: post
title:  "Extending factories"
date:   2015-06-03 11:30:00
author: seth
comments: true
image: /assets/article_images/2015-06-03-modifying-factories/factory.jpg
---

We use [FactoryGirl](https://github.com/thoughtbot/factory_girl) *a lot* in our development team.
In fact we have a whole gem dedicated to providing core business models and their factories.

But sometimes our apps need to extend their functionality. Perhaps it's expecting some extra default values,
it may be extending the models themselves with before and after filters, and so on.

If you tried to redeclare the factory you'd be in for some trouble.

{% highlight ruby %}
# models/organization.rb
class Organization
  key :read_tutorial, Boolean  # we've extended our model to add a new key for this project
end

# factories/organizations.rb
FactoryGirl.define do
  factory :organization do
    read_tutorial   false
  end
end
{% endhighlight %}

```
/factory_girl-4.5.0/lib/factory_girl/decorator.rb:10:in `method_missing':
  Factory already registered: organization (FactoryGirl::DuplicateDefinitionError)
```

### New factory?

At first you may be tempted to just define a new factory. The following code passes.

{% highlight ruby %}
FactoryGirl.define do
  factory :organization_without_tutorial, class: Organization do
    read_tutorial   false
  end
end
{% endhighlight %}

But now we can't use `organization` any more in our app, we have to replace all usage to `organization_without_tutorial`!

### Modify factory!

Fortunately, FactoryGirl offers us a new declaration that we don't often use so it's easy to miss it. `FactoryGirl.modify`

{% highlight ruby %}
FactoryGirl.modify do
  factory :organization do
    read_tutorial   false
  end
end
{% endhighlight %}

Success! It compiles and we're now able to add new functionality to our gem of models and factories.

[factory_girl/#modifying-factories](https://github.com/thoughtbot/factory_girl/blob/master/GETTING_STARTED.md#modifying-factories)