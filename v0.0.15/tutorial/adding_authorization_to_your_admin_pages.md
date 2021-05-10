---
parent: Tutorial
grand_parent: Super v0.0.15
nav_order: 6
---
# Adding authorization to your admin pages

After setting up authenication, you'll need to define a `Controls#initialize`
that accepts an authenticated user. From there, you can customize
`Controls#scope` to have the required behavior.

If you make these changes in your generated `AdminControls`, all of your
`Controls` will inherit your desired behavior.

```ruby
class AdminController < AdminController
  class AdminControls < Super::Controls
    def initialize(current_user)
      @current_user = current_user
    end

    def scope(action:)
      # Example: admins can read and write; others can only read
      if @current_user.admin?
        return model.all
      end

      if action.read?
        return model.all
      end

      raise Super::ClientError::Forbidden
    end
  end

  private

  def new_controls
    Controls.new(current_user)
  end
end
```

