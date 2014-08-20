---
layout: post
title: Deploy a new Sails.js app to Heroku
class: sails-js-heroku
description: It's not quite as easy as deploying a Rails app, but it's still easy.
---

### Pre-requisites
- Set up a Heroku account, install the command line tool and authenticate through it.
- Sails `npm install -g sails@0.1.4` (the latest version as of 20/8/14)

### Basic setup

- Generate your project, cd into the directory: `sails new url-shortener && cd url-shortener`

- Create a Procfile `touch Procfile` then open it and add `web: node app.js`.

- Commit everything to git. `git init && git add . && git commit -m 'Initial commit'`

- Create an app for the project on Heroku: `heroku create brents-url-shortener`

- Set the app environment to production: `heroku config:set NODE_ENV=production`

- Deploy: `git push heroku master`

### Adding some persistence

Next we need to add a MongoDB connection, otherwise we will just be using `sails-disk`
and all of our data will be wiped whenever the app restarts.

- Add the MongoHQ free add-on to the Heroku app: `heroku addons:add mongohq`

- Install the `sails-mongo` adapter: `npm install sails-mongo --save`.

- Heroku automatically creates an environment variable that points to the MongoHQ database when you install the add-on. So just copy the following into `config/connections.js`:

  {% highlight javascript %}
 productionMongoHqDb: {
    adapter: 'sails-mongo',
    url: process.env.MONOGHQ_URL
  }
  {% endhighlight %}

- Change the connection used for models in the production environment to productionMongoHqDb in `config/env/production.js`.

### Use Redis for the session and socket store

In order to share session data between multiple instances, we have to use some shared data store.
Let's use Redis.

- Add the RedisToGo free add-on to the Heroku app: `heroku addons:add redistogo`

- Install the `connect-redis` npm module: `npm install connect-redis@1.4.5 --save-exact`. We specified the version here explicitly because 2.0.0 doesn't seem to work with the current version of Sails.

- Run `heroku config` again and this time we need to pull some information out of the `REDISTOGO_URL` environment variable. We need to set various other variables on Heroku from pieces of this one.

  The url is structured like this:
  `redis://database-name:long-password-here@url.to.the.database.com:port/`

  Use that to set the `REDIS_DB`, `REDIS_PASSWORD`, `REDIS_HOST` and `REDIS_PORT` with `heroku config:set VAR_NAME=value`

- Copy the following into both `config/sessions.js` and `config/sockets.js`, within the `export` object.

  {% highlight javascript %}

 host: process.env.REDIS_HOST,
  port: process.env.REDIS_PORT,
  db: process.env.REDIS_DB,
  pass: process.env.REDIS_PASSWORD
  {% endhighlight %}

- Now so that we can still run our app locally in development, let's make our local environment use the memory store. Paste this into the export object in `config/locals.js`

  {% highlight javascript %}

 session: {
    adapter: 'memory'
  },

  sockets: {
    adapter: 'memory'
  }
  {% endhighlight %}

  Note: I also had to install `sails-disk` again manually in order for `npm install` to work: `npm install sails-disk --save`

- Commit this, and deploy to Heroku again. `git add -A . && git commit -m 'Done!' && git push heroku master`

### Conclusion

Run `heroku logs` and you should see the app start just like when you
run `sails lift` locally. For good measure, you can run `heroku
restart`. You might want to look into
[nodejistu/forever](https://github.com/nodejitsu/forever) if you
actually want to run this for a real app.

It's a bit more difficult than Rails, but pretty straightforward if you
follow this step-by-step - at least on the current version. Thanks to
the SailsCasts folks for their Heroku deployment guide screencast, which
you can find [here](https://www.youtube.com/watch?v=ClHsv81XeaE) -
unfortunately it was created in November, 2013 and Sails has changed in
many ways since then, hence this blog post.

A repository where all of the above changes have been made can be found
at
[brentvatne/url-shortener](https://github.com/brentvatne/url-shortener)
