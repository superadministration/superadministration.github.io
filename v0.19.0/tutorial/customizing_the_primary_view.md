---
parent: Tutorial
grand_parent: Super v0.19.0
nav_order: 4
---
# Customizing the primary view

You can customize your admin controllers by overriding the default definitions of Super's controllers. You can run `bin/rails super:cheat` to see a list of customizable methods. That doesn't tell you how to customize though—let's go through some of the common changes.


## Customizing the `#index` and `#show` views

The method `#display_schema` returns an instance of `Super::Display`. By default, the method first checks which action is calling it, then it looks at the model to see which columns it should display, and in what order.

Let's start at the very beginning. Add this method to your controller (I usually define it as a `private` method), then save and reload.

```ruby
def display_schema
  Super::Display.new do |f, type|
  end
end
```

You now shouldn't see any data columns on the index page, only an "Actions" column. There won't be any data on the show page either.

A quick explainer on the two yielded variables:

* The first yielded value (e.g. `f`) is like a Hash, but not quite. The "hash key" (e.g. `:id` or `:name`) is directly related to the record's method which is often related to the model's table's column.
* The second yielded value (e.g. `type`) denotes how the value should be formatted.
    * `type.string` is the most common type. It calls `to_s` on the return value
    * `type.batch` shows the `to_s` value and a checkbox to use for batch actions.

With that in mind, let's manually make it look closer to what it used to.

```ruby
def display_schema
  Super::Display.new do |f, type|
    f[:id] = type.batch
    f[:name] = type.string
    f[:price_cents] = type.string
  end
end
```

It's a little unnatural to show prices in cents; here where I live, we usually communicate in dollars. So let's try formatting this a bit differently.

```ruby
def display_schema
  Super::Display.new do |f, type|
    f[:id] = type.batch
    f[:name] = type.string
    f[:price_cents] = type.real(:value) { |val| val = val.to_s.rjust(3, "0"); "$#{val[0..-3]}.#{val[-2..-1]}" }
  end
end
```

Super provides two escape hatches here, `type.real` and `type.computed`. You can use `type.real` only for attributes that are ActiveRecord database columns; these are automatically used as sortable columns. You can use `type.computed` in any other circumstance, and these will not be used as database columns.

* These methods accept three different symbols:
    * `:value` yields only the value of the method
    * `:record` yields the entire record. You'll have to call the method yourself
    * `:none` yields nothing
* These methods can return any `String` or `Super::Partial`


### Further customizing the `#show` view

The `#display_schema` method also controls what you see on the show page. If you load any show page now, you should see the exact same data on both the index and show pages. Typically though, show pages tend to have more details than the index page. Let's make it so.

To add a field to only the `#show` page, we'll use the `#current_action` method that's already defined in the controller. With this, we can specify what data to show in the different pages.

```ruby
def display_schema
  Super::Display.new do |f, type|
    f[:id] = type.batch
    f[:name] = type.string
    f[:price_cents] = type.real(:value) { |val| val = val.to_s.rjust(3, "0") ; "$#{val[0..-3]}.#{val[-2..-1]}" }
    if current_action.show?
      f[:order_lines_quantity] = type.computed(:record) { |record| record.order_lines.size }
    end
    f[:created_at] = type.timestamp
    if current_action.show?
      f[:updated_at] = type.timestamp
    end
  end
end
```

Showing a field on only the `#index` page is similar too, you can check for `current_action.index?`.

Note that this is just Ruby, so if you define a method like `#current_admin`, you can show some fields to some users while hiding it for others. You can customize this however you wish!


## Customizing the `#new` and `#edit` views

But it isn't always true! For example, the `#current_action` will claim to be "edit" in the `#update` action when called within the `#form_schema` method.

## Customizing which actions are visible


## Customizing actions behavior

