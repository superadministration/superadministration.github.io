---
parent: Tutorial
grand_parent: Super v0.0.16
nav_order: 7
---
# Adding authorization to your admin pages

One simple way of setting up authorization (access control) is by updating the `#base_scope` method. Every default action that Super defines, like the index or update actions, calls the `#base_scope` action.

If you make these changes in your generated `AdminController`, all of your admin pages will inherit that behavior.

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
