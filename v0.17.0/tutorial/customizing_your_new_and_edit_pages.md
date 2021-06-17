---
parent: Tutorial
grand_parent: Super v0.17.0
nav_order: 5
---
# Customizing your new and edit pages

## Customizing the `#new` form

You can customize the `#new` form by defining `#form_schema` in your admin controller. Let's make it blank.

```ruby
class Admin::ReceiptsController < AdminController
  private

  def model
    Product
  end

  def form_schema
    Super::Form.new do |f, type|
    end
  end
end
```

Once you save and reload that page, you should see a submit button with no form fields.

The `f` variable is similar to a Hash where the key is the column name and the value is the rule we use to turn it into a human readable value.

With that in mind, let's manually make it look like it used to.

```ruby
def form_schema
  Super::Form.new do |f, type|
    f[:customer_id] = type.text_field
    f[:price_cents] = type.text_field
  end
end
```

But this isn't very pleasant to use. It's hard to know what number to type in for the customer ID, so let's make that a dropdown.

```ruby
def form_schema
  Super::Form.new do |f, type|
    customers = Customer.all.map do |c|
      [c.id, c.name]
    end
    f[:customer_id] = type.select(collection: customers)
    f[:price_cents] = type.text_field
  end
end
```

## Customizing the `#edit` form

Let's say for some reason, we only want to let admins set the `price_cents` value when creating a record, but not when editing. We can use the `#current_action` method that's already defined in the admin controller.

```ruby
def form_schema
  Super::Form.new do |f, type|
    f[:customer_id] = type.text_field
    f[:price_cents] = type.text_field(disabled: current_action.edit?)
  end
end
```

This shows the `price_cents` column while preventing it from being edited. You can also hide it altogether by setting `f[:price_cents]` only on the `new?` field.

```ruby
def form_schema
  Super::Form.new do |f, type|
    f[:customer_id] = type.text_field
    if current_action.new?
      f[:price_cents] = type.text_field
    end
  end
end
```

## Customizing strong parameters

You shouldn't have to customize this behavior at all. Super can figure out strong parameters by allowing all of the form fields that are specified as the key `f[$this_part]`; this also works in conjunction with nested attributes.

## Forms with nested attributes

## Using unsupported form fields

## Supported form types

* `#direct` for calling unsupported form builder methods
* `#select`
* `#text_field`
* `#rich_text_area` (ActionText)
* `#check_box`
* `#date_flatpickr`
* `#datetime_flatpickr`
* `#time_flatpickr`
* `#hidden_field`
* `#password_field`
* Nested attributes
    * `#has_many`
    * `#has_one` / `#belongs_to`
    * `#_destroy` checkbox
