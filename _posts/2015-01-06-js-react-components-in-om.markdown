---
layout: post
title: Plain JS React Components in your Om app
class: js-react-om
description: tl;dr - just use it exactly as you would from JavaScript
---

Let's say you want to use an external library such as
[selectize.js](http://brianreavis.github.io/selectize.js/) in your Om
application.  First you'll need to include this library - if you're not
using advanced compilation (you probably should be) then you don't need
externs - but you probably will be eventually (don't worry about it
right now if you're just learning and making toy apps), so generate them
[here](http://www.dotnetwise.com/Code/Externs/) or get externs for a few
common libraries [here](http://closureplease.com/externs/) or just [make
your
own](http://blog.8thlight.com/taryn-sauer/2014/07/31/clojurescript-faux-pas.html).

Selectize is based off of jQuery and doesn't come bundled as a React
component, but you can find a React wrapper for it [here](https://github.com/ggarek/react-selectize).
If there isn't a React wrapper for a componen t you would like to use,
well, you'd have to write your own - perhaps just do it directly in ClojureScript.
But that's not what this post is about.

Now to include the `ReactSelectize` React component within your Om app
(on `0.8.0-rc1` which is based on React `0.12.2` - you'll have to modify
this slightly for older React versions because `createElement` is new),
let's add this `react-utils.core` module:

  {% highlight clojure %}
  (ns react-util.core)

  (defn build [component props]
    (let [React (.-React js/window)]
      (.createElement React component (clj->js props))))
  {% endhighlight %}

And inside of our Om app we just have to require `react-util.core` and
call the build function with our React component as the first argument
and the component's props as the second argument:

  {% highlight clojure %}
  (ns example.core
    (:require [om.core :as om]
              [om-tools.dom :as dom :include-macros true]
              [om-tools.core :refer-macros [defcomponent]]
              [react-util.core :as react]]))

  (defcomponent my-component
    [app-state owner]
    (render [_]
     (react/build (.-ReactSelectize js/window)
                  {:selectId "my-select-id" :onChange #(.log js/console %)})))
  {% endhighlight %}

So easy! Thanks to David Nolen for pointing out the obvious solution -
it just works.

> _dnolen_: making sure that all this stuff works was/is priority #1
