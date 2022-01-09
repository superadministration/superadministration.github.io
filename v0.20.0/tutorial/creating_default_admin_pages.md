---
parent: Tutorial
grand_parent: Super v0.20.0
nav_order: 3
---
# Creating default admin pages

Super comes with generators for generating admin routes and controllers.

```sh
bin/rails g super:resource product
```

That'll update the routes file and generate a controller file like below:

```ruby
# config/routes.rb
Rails.application.routes.draw do
  namespace :admin do
    resources :products
  end
end

# app/controllers/admin/products_controller.rb
class Admin::ProductsController < AdminController
  private

  def model
    Product
  end
end
```

Let's create two more using the generator.

```sh
bin/rails g super:resource customer
bin/rails g super:resource order
```

Start the rails server and navigate to <http://localhost:3000/admin/products> and see what the defaults look like.


## The generated routes

You may have noticed that <http://localhost:3000/admin> gives you a 404 error. You can create something like a `Admin::HomeController` and customize it to your liking, but for now, let's redirect it to a reasonable page. I'll make it redirect to the customers page.

While we're here, let's also change the order of the resources. By default, Super uses the order of the routes to build the navigation links.

```ruby
Rails.application.routes.draw do
  namespace :admin do
    resources :orders
    resources :customers
    resources :products

    root to: redirect(path: "/admin/orders", status: 302)
  end
end
```
