---
layout: post
title: Angular.js with Flux - Async Requests
class: angular-flux-async
description: Detailing some experimentation I have done using the Flux architecture pattern with Angular.js
---

I've never quite felt comfortable with the patterns for data flow in
Angular apps, especially around asynchronous data.

There has been a lot of hype around React and the data flow architecture
pattern that has emerged arround it called
[Flux](http://facebook.github.io/react/docs/flux-overview.html), and
while I'm not quite ready to jump in to React for most of my day-to-day
work due to its relative immaturity compared to Angular (forms,
validations, animations, aria and lack of other libraries would slow me
down in React), I became curious about how I might be able to adapt Flux
to the Angular context.

I'd highly recommend reading [the Flux
documentation](https://facebook.github.io/flux/docs/overview.html#content)
and watching the [egghead.io series on the React - Flux
architecture](https://egghead.io/series/react-flux-architecture) - most
of this post won't make sense without that background information.

So, back to Angular. Thankfully, [others had the idea of using Flux with
angular before I did, and they wrote about
it](http://victorsavkin.com/post/99998937651/building-angular-apps-using-flux-architecture).
I still wasn't quite satisfied, though - my first incliation was make
API calls to fetch and update data directly from the store, which I
quickly found to be very awkward.

For example:

  {% highlight javascript %}
    Store.fetchRecord(id).then(function(record) { $scope.record = record })
  {% endhighlight %}

And I also want to make sure I keep this updated with changes, so:

  {% highlight javascript %}
    Store.on('change', function(action) { $scope.record = Store.getRecord() })
  {% endhighlight %}

And `Store.fetchRecord` would look something like:

  {% highlight javascript %}
    $http.get("/records/#{id}").then(function(response) { _record = response.data})
  {% endhighlight %}

Lastly, `Store.getRecord` is simply this:

  {% highlight javascript %}
    return _record;
  {% endhighlight %}

Easy enough, although it certainly looks a bit complicated for just
fetching some data.

But let's say we were to add on some more robust functionality: handling
timeouts, errors, rejected auth tokens. Suddenly we're loading the store
with all sorts of API related concerns. Wouldn't it be easier if the
store only dealt with synchronous operations?

Inspired by [this insightful blogpost called "Async requests with
React.js and Flux, revisited."](http://www.code-experience.com/async-requests-with-react-js-and-flux-revisited/)
I tried out at an approach that does just that: isolates asynchronous
code to API modules, and has stores only contain synchronous code,
described in the following diagram.

![Diagram showing the above description visually](https://raw.githubusercontent.com/brentvatne/brentvatne.github.io/master/images/angular-flux-async.png)

I still haven't decided whether this is an approach I will continue to
use, but in the limited experience I've had with it, I find that it
makes the state of the application easier to reason about. You can try
doing this yourself with a little library I'm casually working on (as
in YMMV if you choose to use this in production) called
[angular-flux](https://github.com/brentvatne/angular-flux) - it just
packages up `dispatcher.js` and provides some utility functions for
reducing some of the boilerplate associated with setting up stores,
actions and bindings.
