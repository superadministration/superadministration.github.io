---
parent: Tutorial
grand_parent: Super v0.22.0
nav_order: 4
---
# Customizing the primary view

You can customize your admin controllers by overriding the default definitions of Super's controllers. You can run `bin/rails super:cheat` to see a list of customizable methods. That doesn't tell you how to customize though—let's go through some of the common changes.


## Customizing the `#index` and `#show` views

The method `#display_schema` returns an instance of `Super::Display`. By default, the method first checks which action is calling it, then it looks at the model to see which columns it should display, and in what order.

Let's start at the very beginning. Add this method to your controller (I usually define it as a `private` method), then save and reload.

```ruby
private

def model
  Product
end

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

### Using unsupported display types

It's a little unnatural to show prices in cents; here where I live, we usually communicate in dollars. So let's try formatting this a bit differently.

```ruby
def display_schema
  Super::Display.new do |f, type|
    f[:id] = type.batch
    f[:name] = type.string
    f[:price_cents] = type.real(:column) { |val| val = val.to_s.rjust(3, "0"); "$#{val[0..-3]}.#{val[-2..-1]}" }
  end
end
```

Super provides two escape hatches here, `type.real` and `type.computed`. You can use `type.real` only for attributes that are ActiveRecord database columns; these are automatically used as sortable columns. You can use `type.computed` in any other circumstance, and these will not be used as database columns.

* These methods accept three different symbols:
    * `:column` yields only the value of the method
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
    f[:price_cents] = type.real(:column) { |val| val = val.to_s.rjust(3, "0") ; "$#{val[0..-3]}.#{val[-2..-1]}" }
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


### Using `Super::Badge`

You can specify "badges" or "pills" by returning an instance of `Super::Badge`.

```ruby
def display_schema
  Super::Display.new do |f, type|
    f[:id] = type.batch
    f[:status] = type.computed(:record) do |record|
      if record.updated_at == record.created_at
        Super::Badge.new("Never updated", styles: :green)
      elsif record.updated_at > record.created_at
        Super::Badge.new("Updated", styles: :yellow)
      else
        Super::Badge.new("Invalid update", styles: :red)
      end
    end
  end
end
```

The full list of styles are:

* `light`
* `dark`
* `red`
* `yellow`
* `green`
* `blue`
* `purple`


## Customizing the `#new` and `#edit` views

The `#form_schema` method returns an instance of `Super::Form`.

Again, let's start with an empty form.

```ruby
private

def model
  Order
end

def form_schema
  Super::Form.new do |f, type|
  end
end
```

The form `type`s match fairly closely to what you might be used to with Rails' `form_for` methods. Here's an example.

```ruby
def form_schema
  Super::Form.new do |f, type|
    f[:customer_id] = type.text_field
    # In a `form_for`, the above line would probably look like:
    # f.text_field :customer_id
  end
end
```

It's not very convenient to type in the customer IDs every time though.

```ruby
def form_schema
  customers = Customer.all.map do |c|
    [c.name, c.id]
  end

  Super::Form.new do |f, type|
    f[:customer_id] = type.select(customers)
  end
end
```

### Forms with nested attributes

Let's make it so that we can edit order lines from within the order.

```ruby
def form_schema
  customers = Customer.all.map do |c|
    [c.name, c.id]
  end
  products = Product.all.map do |p|
    [p.name, p.id]
  end

  Super::Form.new do |f, type|
    f[:customer_id] = type.select(customers)
    f[:order_lines_attributes] = type.has_many(:order_lines) do |g|
      g[:product_id] = type.select(products)
      g[:_destroy] = type._destroy
    end
  end
end
```

In addition to `type.has_many`, there's also `type.has_one` and `type.belongs_to`.

### Using unsupported form fields

You have two options here.

* `type.direct(:text_field, super_method: true)`
    * The first argument denotes which form builder method to call.
    * The `super_method` keyword:
        * When `true`, calls the Super builder (e.g. `f.super.text_field`)
        * When `false`, calls the default builder (e.g. `f.text_field`)
    * Any additional arguments and keyword arguments gets passed directly to the form builder method
* `type.partial("partial")` lets you use a custom partial
    * The partial defines the variable `form`, which is the yielded value of Rails' `form_for`
    * You can place partials under `app/views/admin/#{your_resource}/_#{partial}.html.erb`


### Having different edit and new forms

Here we can use `current_action.new?` and `current_action.edit?`.

But `current_action` isn't always true! For example, the `#current_action` will claim to be "edit" in the `#update` action when called within the `#form_schema` method. (The same thing happens in the `#create` action.)

This is for convenience, so that you don't need to check `current_action.new? || current_action.create?`.


### Customizing strong parameters

You shouldn't have to customize this behavior at all. Super can figure out strong parameters by allowing all of the form fields that are specified as the key `f[$this_part]`. This also works in conjunction with nested attributes.
