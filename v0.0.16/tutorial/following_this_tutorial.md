---
parent: Tutorial
grand_parent: Super v0.0.16
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

The SAAS app will keep track of **Products**, **Customers**, **Receipts**, and **ReceiptLineItems**.

```sh
bin/rails g model product name:string price_cents:integer
bin/rails g model customer name:string
bin/rails g model receipt customer:references total_cents:integer
bin/rails g model receipt_line_item receipt:references product:references
bin/rails db:migrate
```

We'll need to define some relations and validations too.

```ruby
class Receipt < ApplicationRecord
  belongs_to :customer
  has_many :receipt_line_items
end

class Customer < ApplicationRecord
  has_many :receipts

  validates :name, presence: true
end
```

Let's create some fake data using the Rails console.

```ruby
# bin/rails console
ActiveRecord::Base.transaction do
  calculator = Product.create!(name: "Graphic calculator", price_cents: 100_00)
  cheeseburger = Product.create!(name: "Cheeseburger", price_cents: 6_00)
  magazine = Product.create!(name: "Magazine", price_cents: 3_00)
  sneakers = Product.create!(name: "Sneakers", price_cents: 55_00)
  tote = Product.create!(name: "Tote", price_cents: 20_00)
  products = [calculator, cheeseburger, magazine, sneakers, tote]

  (1..1000).each do |i|
    name = %w[Alice Bob Carol Dan][i % 4]
    Customer.create!(name: "#{name} #{i}")
  end

  Customer.find_each do |customer|
    product_index_start = rand(products.size)
    product_index_end = rand(product_index_start..products.size)
    receipt = Receipt.new(customer: customer)
    total_cents = 0
    (product_index_start...product_index_end).each do |i|
      receipt_line_item = receipt.receipt_line_items.build(product: products.fetch(i))
      total_cents += receipt_line_item.product.price_cents
    end
    receipt.total_cents = total_cents
    receipt.save!
  end
end
```


## Set up Super

I'll [set up Super](installation_and_setup.md) using the default settings.

Let's begin!
