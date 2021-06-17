---
parent: Tutorial
grand_parent: Super v0.17.0
nav_order: 4
---
# Customizing your index and show pages

## Customizing the `#index` page

After generating the controller, we'll need to update it by adding method called `#display_schema`. It'll return an instance of `Super::Display`. Let's start with `Admin::ProductsController`.

```ruby
class Admin::ProductsController < AdminController
  private

  def model
    Product
  end

  def display_schema
    Super::Display.new do |f, type|
    end
  end
end
```

Once you save that and reload the page, you won't see any data columns, only an "Actions" column.

The `f` variable is similar to a Hash where the key is the column name and the value is the rule we use to turn it into a human readable value.

With that in mind, let's manually make it look like it used to.

```ruby
def display_schema
  Super::Display.new do |f, type|
    f[:name] = type.string
    f[:price_cents] = type.string
  end
end
```

We see there that strangely, that the type for `price_cents` is `string` and not `integer`. The `string` here should be thought of as `to_s`, not the actual type of the column.

It's a little unnatural to show prices in cents; here where I live, we usually communicate in dollars. So let's try formatting this a bit differently.

```ruby
def display_schema
  Super::Display.new do |f, type|
    f[:name] = type.string
    f[:price_cents] = type.real { |column_value| "$#{sprintf("%.2f", column_value)}" }
  end
end
```

By choosing `type.real`, we're telling Super that `:price_cents` is a database column and is thus filterable.

You'll notice though that the name of the column still reads `Price cents`. Here's one way we can get around that.

```ruby
def display_schema
  Super::Display.new do |f, type|
    f[:name] = type.string
    f[:price] = type.computed(:record) { |record| "$#{sprintf("%.2f", record.price_cents)}" }
  end
end
```

Since we've specified `type.computed` here, Super knows not to attempt to search using the `price` column. Both `computed` and `real` accept arguments which tell it what to yield into your block: `:column` is the default and it yields the value of the column (it doesn't have to be a column, just a method on the object), `:record` yields the entire record itself, and `:none` doesn't yield anything.

## Customizing the `#show` page

Customizing this page is identical to the `#index` page. However, `#index` pages tend to highlight some important fields while the `#show` page might show every field.

Let's start with a controller that already has the `#index` view set up.

```ruby
class Admin::CustomersController < AdminController
  private

  def model
    Customer
  end

  def display_schema
    Super::Display.new do |f, type|
      f[:name] = type.string
      if current_action.show?
        f[:receipt_count] = f.computed(:record) { |record| record.receipts.size }
      end
    end
  end
end
```

To add a field to only the `#show` page, we'll use the `#current_action` method that's already defined in the controller. With this, we can specify what data to show in the different pages.

```ruby
def display_schema
  Super::Display.new do |f, type|
    f[:name] = type.string
    if current_action.show?
      f[:receipt_count] = f.computed(:record) { |record| record.receipts.size }
    end
  end
end
```

Showing a field on only the `#index` page is similar too, you can check for `current_action.index?`.

Note that this is just Ruby, so if you define a method like `#current_admin`, you can show some fields to some users while hiding it for others. You can customize this however you wish!

## Supported display types

* `#real`
* `#computed`
* `#string`
* `#timestamp`
* `#time`
* `#rich_text` (ActionText)
* `#badge`
* `#actions`
