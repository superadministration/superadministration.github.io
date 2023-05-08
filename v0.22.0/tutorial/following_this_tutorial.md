---
parent: Tutorial
grand_parent: Super v0.22.0
nav_order: 2
---
# Following this tutorial

We're going to build some admin pages for an e-commerce site. Feel free to follow along step by step, but you don't have to.


## Create your Rails app

I'll be working on a Rails app created like this:

```sh
rails new super_tutorial --database=sqlite3
```


## Create some database models to work with

The SAAS app will keep track of **Products**, **Customers**, **Orders**, and **OrderLines**.

```sh
bin/rails g model customer name:string
bin/rails g model product name:string price_cents:integer
bin/rails g model order customer:references
bin/rails g model order_line order:references product:references
bin/rails db:migrate
```

We'll need to define some relations and validations on the model. The meaning of these is beyond the scope of this tutorial, but the official Rails documentation goes into a good amount of detail.

```ruby
class Customer < ApplicationRecord
  has_many :orders, dependent: :destroy
  validates :name, presence: true
end

class Product < ApplicationRecord
  validates :name, presence: true
  validates :price_cents, presence: true, numericality: { only_integer: true, greater_than_or_equal_to: 0 }
end

class Order < ApplicationRecord
  belongs_to :customer
  has_many :order_lines, dependent: :destroy
  has_many :products, through: :order_lines
  accepts_nested_attributes_for :order_lines, reject_if: -> (attrs) { attrs["product_id"].blank? }
end

class OrderLine < ApplicationRecord
  belongs_to :order
  belongs_to :product
end
```

Let's create some fake data using the Rails console.

```ruby
# bin/rails console
srand 139
ActiveRecord::Base.transaction do
  calculator = Product.create!(name: "Graphing calculator", price_cents: 100_00)
  cheeseburger = Product.create!(name: "Cheeseburger", price_cents: 6_00)
  magazine = Product.create!(name: "Magazine", price_cents: 3_00)
  sneakers = Product.create!(name: "Sneakers", price_cents: 55_00)
  tote = Product.create!(name: "Tote", price_cents: 20_00)
  products = [calculator, cheeseburger, magazine, sneakers, tote]

  (1..1000).each do |i|
    puts "Creating Customers #{i} to #{i + 99}" if i % 100 == 1
    name = %w[Alice Bob Carol Dan][i % 4]
    Customer.create!(name: "#{name} #{i}")
  end

  Customer.order(:id).find_each do |customer|
    puts "Creating Orders for Customers #{customer.id} to #{customer.id + 99}" if customer.id % 100 == 1
    number_of_orders = rand(1..3)
    number_of_orders.times do
      order = customer.orders.build
      order_products = products.sample(rand(1..products.size))
      order_products.each do |product|
        order.order_lines.build(product: product)
      end
      order.save!
    end
  end
end
```


## Set up Super

I'll [set up Super](./installation_and_setup.md) using the default settings.

Let's begin!
