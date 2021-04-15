---
parent: Tutorial
grand_parent: Super v0.0.13
nav_order: 4
---
# Customizing your admin pages

## Customers

We won't make any customizations in the admin page, but let's add a validation
to the Customer model.

```ruby
# app/models/customer.rb
class Customer < ApplicationRecord
  validates :name, presence: true
end
```

## Products

Let's format the prices, and on the show page, let's add the number of current
subscriptions.

```ruby
# In app/models/product.rb
class Product < ApplicationRecord
  has_many :subscriptions
  monetize :price_cents # this is a feature of rails-money
end
```

The interesting thing to keep in mind here are the calls to `type.computed`.
The call to `computed` tells Super that you're defining a field that isn't a
column. The first argument, `:column` yields the value of that field/method,
while `:record` yields the entire record. The value of the block is what is
shown to the user.

The call to `type.string` might seem a little incorrect, but you can think of it
like calling `#to_s` on the field to get it ready for display.

```ruby
# In app/controllers/admin/products_controller.rb
class Admin::ProductsController < AdminController
  class Controls < AdminControls
    def model
      Product
    end

    def display_schema(action:)
      Super::Display.new do |f, type|
        f[:id] = type.string
        f[:name] = type.string
        f[:price] = type.computed(:column, &:format)
        if action.show?
          time = Time.current
          f[:current_subscriptions] = type.computed(:record) do |record|
            record.subscriptions.where("starts_at < ? AND ends_at > ?", time, time).size
          end
          f[:created_at] = type.timestamp
          f[:updated_at] = type.timestamp
        end
      end
    end
  end
end
```

Note that this doesn't change the filtering or sorting sections. That's
customizable too, but we'll get to that later.

The index view:

![](/screenshots/0-0-12/products_custom1_index.png)

The show view:

![](/screenshots/0-0-12/products_custom1_show.png)


## Receipts

Let's format the prices here too, but for good measure, we'll do it a little
more manually without editing the model. We'll also add a link to the associated
subscription.

Note the `type.real` here. It's works identically to `type.computed` except that
it tells Super that it is a real database column.

```ruby
# app/controllers/admin/receipts_controller.rb
class Admin::ReceiptsController < AdminController
  class Controls < AdminControls
    def model
      Receipt
    end

    def display_schema(action:)
      Super::Display.new do |f, type|
        f[:id] = type.string
        f[:total] = type.computed(:record) do |record|
          Money.new(record.total_cents, record.total_currency).format
        end
        f[:subscription_id] = type.real(:column) do |value|
          AdminController.helpers.link_to(
            "Subscription ##{value}",
            Rails.application.routes.url_helpers.admin_subscription_path(value)
          )
        end
        f[:created_at] = type.timestamp
        if action.show?
          f[:updated_at] = type.timestamp
        end
      end
    end
  end
end
```

The index view:

![](/screenshots/0-0-12/receipts_custom1_index.png)

The show view:

![](/screenshots/0-0-12/receipts_custom1_show.png)


## Subscriptions

We'll do a little more here. In addition to showing the related Product and
Customer, let's show the if the subscription is currently active.

We'll also update the forms and turn the product field into a dropdown. And
while we're at it, let's use nested attributes so that we can edit or create a
customer here.

```ruby
# app/models/subscription.rb
class Subscription < ApplicationRecord
  belongs_to :product
  belongs_to :customer

  accepts_nested_attributes_for :customer
  validates_associated :customer
end
```

The interesting thing to note here is the nested attributes. Note that the
`f[:customer_attributes]` is what'll be set, while the `type.has_one(:customer)`
is what'll be read. Also of note, `type.has_one` and `type.belongs_to` are
aliases.

```ruby
# app/controllers/admin/subscriptions_controller.rb
class Admin::SubscriptionsController < AdminController
  class Controls < Super::Controls
    def model
      Subscription
    end

    def scope(action:)
      model.includes(:customer, :product)
    end

    def display_schema(action:)
      current_time = Time.current
      Super::Display.new do |f, type|
        f[:id] = type.string
        f[:customer_id] = type.real(:record) do |record|
          customer = record.customer
          AdminController.helpers.link_to(
            customer.name,
            Rails.application.routes.url_helpers.admin_customer_path(customer.id)
          )
        end
        f[:product_id] = type.real(:record) do |record|
          product = record.product
          AdminController.helpers.link_to(
            product.name,
            Rails.application.routes.url_helpers.admin_product_path(product.id)
          )
        end
        f[:status] = type.computed(:record) do |record|
          if record.starts_at.nil? || record.ends_at.nil?
            "Unknown"
          elsif record.starts_at <= current_time && record.ends_at >= current_time
            "Active"
          elsif action.show?
            "Inactive"
          end
        end
        if action.index?
          f[:created_at] = type.timestamp
        else
          f[:starts_at] = type.timestamp
          f[:ends_at] = type.timestamp
          f[:created_at] = type.timestamp
          f[:updated_at] = type.timestamp
        end
      end
    end

    def form_schema(action:)
      Super::Form.new do |f, type|
        f[:customer_attributes] = type.has_one(:customer) do
          f[:name] = type.string
        end
        f[:product_id] = type.select(collection: Product.all.map { |product| [product.name, product.id] })
        f[:starts_at] = type.string
        f[:ends_at] = type.string
      end
    end
  end
end
```

The index view:

![](/screenshots/0-0-12/subscriptions_custom1_index.png)

The show view:

![](/screenshots/0-0-12/subscriptions_custom1_show.png)

The edit view:

![](/screenshots/0-0-12/subscriptions_custom1_edit.png)

The new view (with a validation error):

![](/screenshots/0-0-12/subscriptions_custom1_new.png)
