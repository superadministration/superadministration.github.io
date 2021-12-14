---
parent: Guides
grand_parent: Super v0.19.0
nav_order: 4
---
# Using ActionText

Super has some basic support for editing and viewing ActionText. Since
ActionText only works with Webpacker, you'll have to set up Webpacker for your
application. (You don't have to configure Super to not use Sprockets.)

Once you've set that up, ActionText should be relatively simple. The following
command will copy over the necessary JS and CSS into your application:

```bash
bundle exec rails generate super:action_text
```

As an example, let's say we're making admin pages for a blog. We'll keep this
example simple and work with a `Post` model with just a `#title` column and
a rich text `#content` field.

```ruby
class Post < ApplicationRecord
  has_rich_text :content
end
```

We'll need to configure the `#form_schema` and `#display_schema` methods.

```ruby
class Controls < Super::Controls
  # ...

  def form_schema(action:)
    Super::Form.new do |fields, type|
      fields[:title] = type.text
      fields[:content] = type.rich_text_area
    end
  end

  def display_schema(action:)
    Super::Display.new do |fields, type|
      fields[:title] = type.text
      fields[:content] = type.rich_text
    end
  end
end
```
