---
parent: Tutorial
grand_parent: Super v0.17.0
nav_order: 3
---
# Creating default admin pages

## Using the generator

Super comes with generators for generating admin routes and controllers.

```sh
bin/rails g super:resource product
```

That'll update the routes file and generate a controller file like below:

```ruby
# app/controllers/admin/products_controller.rb
class Admin::ProductsController < AdminController
  private

  def model
    Product
  end
end

# config/routes.rb
Rails.application.routes.draw do
  namespace :admin do
    resources :products
  end
end
```

Before we continue, we'll create the rest of the admin pages too.

```sh
bin/rails g super:resource customer
bin/rails g super:resource receipt
```

Let's start the rails server and navigate to <http://localhost:3000/admin/products> and see what the defaults look like.

The index view:

![](/screenshots/0-0-12/products_default_index.png)

The show view:

![](/screenshots/0-0-12/products_default_show.png)

The edit view:

![](/screenshots/0-0-12/products_default_edit.png)

## The generated routes

You may have noticed that <http://localhost:3000/admin> gives you a 404 error. You can create something like a `Admin::HomeController` and customize it to your liking, but for now, let's redirect it to a reasonable page. I'll make it redirect to the customers page.

While we're here, let's also change the order of the resources. By default, Super uses the order of the routes to build the navigation links.

```ruby
Rails.application.routes.draw do
  namespace :admin do
    resources :customers
    resources :receipts
    resources :products

    root to: redirect(path: "/admin/customers", status: 302)
  end
end
```
