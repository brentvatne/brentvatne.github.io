---
layout: post
title: Using dotenv to manage sensitive information
class: dotenv-howto
description: Store API credentials such as Stripe and Amazon S3 securely
in a local config file called .env with the help of a little gem.
---

Stripe and Amazon S3 credentials are a couple of common
examples of sensitive information that you will frequently store
in your app. I've found that the easiest way to store / access
this information in your app is through (environment variables)[http://www.wikiwand.com/en/Environment_variable],
and that the best way to manage these values across various apps
is with `dotenv` - specifically with `dotenv-rails` for Rails
apps.

Just add `dotenv-rails` to your Gemfile

```ruby
# In Gemfile
gem 'dotenv-rails', groups: [:development, :test]
```

Create a file named `.env` in your project root and set any variable in it
like so:

```ruby
# In .env
VARIABLE_NAME=some-value-here
```

And lastly, add this to your .gitignore so that you do not commit your
super secret credentials.

```ruby
# In .gitignore
.env
```

If you are using Heroku, setting the environment variables on your
server is as simple as `heroku config:set VARIABLE_NAME=value`, or you
can just use the (heroku-config)[https://github.com/ddollar/heroku-config]
gem to push your local environment variables defined in `.env` with
`heroku config:push` (see more details in the Gem docs).

If you are using a VPS, such as DigitalOcean, you will have to configure
the environment variables on the server in a different way. For example,
with Unicorn, modify `/etc/default/unicorn` and (add your config
there)[https://www.digitalocean.com/community/questions/unicorn-not-reading-environment-variables-correctly].
