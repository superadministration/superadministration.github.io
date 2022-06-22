---
parent: Tutorial
grand_parent: Super v0.21.0
nav_order: 1
---
# Installation and setup

Add this line to your application's Gemfile:

```ruby
gem "super"
```

And then execute:

```sh
bundle install
```

Next, you'll need to add a config file and some of the base files by running this command:

```sh
bin/rails generate super:install
```

The default configuration makes a few assumptions, but it is configurable. See [installation guide](../guides/installation_options.md) to learn more.
