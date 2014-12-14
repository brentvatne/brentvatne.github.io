---
layout: post
title: Angular.js with Flux: Async Requests
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

`Store.fetchRecord(id).then((record) -> $scope.record = record)`

And I also want to make sure I keep this updated with changes, so:

`Store.on('change', -> $scope.record = Store.getRecord())`

And `Store.fetchRecord` would look something like:

`$http.get("/records/#{id}").then((response) -> _record = response.data)`

Easy enough, although it certainly looks a bit complicated for just
fetching some data. But let's say we were to add on some more robust
functionality: handling timeouts, errors, rejected auth tokens. Suddenly
we're loading the store with all sorts of API related concerns. Wouldn't
it be easier if the store itself only dealt with synchronous operations?

Inspired by [this insightful blogpost called "Async requests with
React.js and Flux, revisited."](http://www.code-experience.com/async-requests-with-react-js-and-flux-revisited/)
I tried out at an approach that did just that: isolated asynchronous
code to API modules, and had stores only contain synchronous code.

- When I want to update the state, for example in a directive
  controller, I call `Actions.fetchRecord(id)`
- In that same controller I watch the record for the change event
  and update when it is changed - essentially the same as above.
- The API call is performed by the action creator directly, and
  only when the API call is complete will the action be dispatched,
  along with the result. The API service can handle exceptional cases /
  retries.
- The Store handles the dispatched action by simply persisting the state
  locally and firing the change event.

But hey, maybe this visual will help:
![https://raw.githubusercontent.com/brentvatne/brentvatne.github.io/master/images/angular-flux-async.png]

I still haven't decided whether this is an approach I will continue to
use, but in the limited experience I've had with it, I find that it
makes the state of the application easier to reason about. You can try
doing this yourself with a little library I'm casually working on called
[angular-flux](https://github.com/brentvatne/angular-flux) - it just
packages up dispatcher.js and provides some useful utility functions for
skipping over some of the boilerplate.
