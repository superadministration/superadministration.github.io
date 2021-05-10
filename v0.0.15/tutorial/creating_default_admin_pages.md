---
parent: Tutorial
grand_parent: Super v0.0.15
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
class Admin::ProductsController < AdminController
  class Controls < AdminControls
    def model
      Product
    end
  end
end
```

Let's start the rails server and browse over to
<http://localhost:3000/admin/products> and see what the defaults look like.

The index view:

![](/screenshots/0-0-12/products_default_index.png)

The show view:

![](/screenshots/0-0-12/products_default_show.png)

The edit view:

![](/screenshots/0-0-12/products_default_edit.png)

Let's create the rest of the admin pages too.

```sh
bin/rails g super:resource customer
bin/rails g super:resource subscription
bin/rails g super:resource receipt
```

## The generated routes

You may have noticed that <http://localhost:3000/admin> gives you a 404 error.
You can create something like a `Admin::HomeController` and customize it to your
liking, but for now, let's redirect it to a reasonable page. I'll make it
redirect to the customers page.

```ruby
Rails.application.routes.draw do
  namespace :admin do
    resources :receipts
    resources :subscriptions
    resources :customers
    resources :products

    root to: redirect(path: "/admin/customers", status: 302)
  end
  # For details on the DSL available within this file, see https://guides.rubyonrails.org/routing.html
end
```

While we're here, let's change the order of the resources. Super uses the order
of the routes to build the navigation links. And lastly, I'm gonna remove the
comment.

```ruby
Rails.application.routes.draw do
  namespace :admin do
    resources :customers
    resources :subscriptions
    resources :receipts
    resources :products

    root to: redirect(path: "/admin/customers", status: 302)
  end
end
```
