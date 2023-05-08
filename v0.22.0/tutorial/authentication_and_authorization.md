---
parent: Tutorial
grand_parent: Super v0.22.0
nav_order: 5
---
# Authentication and authorization

Since Super builds on standard Rails controllers, it's fairly simple to setup authentication and authorization however you like.

(Authentication is related to logins, while authorization is related to permissions.)

## Authentication


## Authorization

My preferred way of setting up authorization is by updating the `#base_scope` method. Every default action that Super defines, like the index or update actions, calls the `#base_scope` action.

If you make these changes in your generated `AdminController`, all of your admin pages will inherit that behavior. You can then further override that in your individual controllers, if necessary.

```ruby
class AdminController < AdminController
  private

  def base_scope
    # Example: admins can read and write; others can only read
    if current_user.admin?
      return model.all
    end

    if current_action.read?
      return model.all
    end

    raise Super::ClientError::Forbidden
  end
end
```

Note that the correct behavior is to `raise` an error, not to return `model.none`. If you use `model.none`, users will be able to create new records (but won't be able to edit). Even if you do want this behavior, I recommend making it explicit and using `raise`.
