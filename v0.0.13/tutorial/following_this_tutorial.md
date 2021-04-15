---
parent: Tutorial
grand_parent: Super v0.0.13
nav_order: 2
---
# Following this tutorial

We're going to build some admin pages for a pretend SAAS product. Feel free to follow along step by step, but you don't have to.


## Create your Rails app

I'll be working on a Rails app created like this:

```sh
rails new super_saas --database=sqlite3
```


## Create some database models to work with

The SAAS app will keep track **Products**, **Customers**, **Subscriptions**, and **Receipts**.

```sh
bin/rails g model product name:string price_cents:integer price_currency:string
bin/rails g model customer name:string
bin/rails g model subscription product:references customer:references starts_at:datetime ends_at:datetime
bin/rails g model receipt subscription:references total_cents:integer total_currency:string
bin/rails db:migrate
```

I'll create some fake data using the Rails console.

```ruby
# bin/rails console
starter = Product.create!(name: "Starter", price_cents: 10_00, price_currency: "USD")
basic = Product.create!(name: "Basic", price_cents: 20_00, price_currency: "USD")
premium = Product.create!(name: "Premium", price_cents: 30_00, price_currency: "USD")
business = Product.create!(name: "Business", price_cents: 40_00, price_currency: "USD")
enterprise = Product.create!(name: "Enterprise", price_cents: 50_00, price_currency: "USD")
ultimate = Product.create!(name: "Ultimate", price_cents: 60_00, price_currency: "USD")
Customer.transaction do
  (1..1000).each do |i|
    name = %w[Alice Bob Carol Dan][i % 4]
    Customer.create!(name: "#{name} #{i}")
  end
end
Subscription.transaction do
  products = [starter, basic, premium, business, enterprise, ultimate]
  beginning_of_day = Time.current.beginning_of_day
  Customer.find_each do |c|
    start = beginning_of_day + rand(-2..2).months
    sub = Subscription.create!(
      product: products.sample,
      customer: c,
      starts_at: start,
      ends_at: start + rand(1..3).months
    )

    Receipt.create!(
      subscription: sub,
      total_cents: sub.product.price_cents,
      total_currency: sub.product.price_currency,
    )
  end
end
```


## Set up various libraries

I'll [set up Super](installation_and_setup.md) using the default settings.

I'll also install [`money-rails`](https://github.com/RubyMoney/money-rails) to handle some of the money stuff.

Let's begin!
