---
parent: Guides
grand_parent: Super v0.18.0
nav_order: 2
---
# Using Webpacker instead of Sprockets

Super defaults to using Sprockets and using Webpacker only when necessary.

It's also possible to configure Super to use Webpacker exclusively. **I do not
recommend this since it's slower, harder to use, and prints warnings.** But it's
supported and totally possible to do if you want to or if you need to.

You'll first need to set up Webpacker to handle ERB templates.

After installing Super, run one the following:

## Webpacker 4 and 5

```bash
bundle exec rails webpacker:install:erb # if you haven't already
bundle exec rails generate super:webpacker
```

## Webpacker 6

```bash
yarn add rails-erb-loader # if you haven't already
bundle exec rails generate super:webpacker
```
