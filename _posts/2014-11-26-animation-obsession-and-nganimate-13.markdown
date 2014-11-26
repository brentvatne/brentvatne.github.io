---
layout: post
title: Recent animations obsession & notes about ngAnimate
class: animations-ng-animate-13-intro
description: Recently I've found myself extremely interested in animations. In this post I talk about my learning journey so far, and a bit about ngAnimate in Angular 1.3.
---

Recently I've found myself extremely interested in animations. My interest
reached a boiling point when Google released Material Design ("Motion Provides
Meaning"), but animations have been making a big push into mobile and web app
development for quite some time now.  Animations bring life and a real sense of
enjoyment to using apps, and I just had to learn about how to use them best in
my work. Android 5 is an example of seamless, performant and fun animations
that make the experience of using my phone better than ever before.

I started looking first at [Ionic Framework](http://ionicframework.com/) and
being impressed but frustrated by performance on mobile. Most animations within the
framework work really great on desktop Chrome, but jank sets in pretty quickly
when you boot it up on Android, so you have to be very conservative in what
animations you choose to apply. That said, [this is a pretty neat
component](http://ionicframework.com/blog/tinder-for-x/).

So I went ahead and read [Google's Web Fundamentals on
Animations](https://developers.google.com/web/fundamentals/look-and-feel/animations/),
a very concise primer on creating animations for the web. I'd recommend this to
anyone before moving forward.

Next I just [experimented with re-creating some animations that I
liked](https://github.com/brentvatne/web-experiments), such as the [card drag
animation from Google
Now](https://github.com/brentvatne/web-experiments/blob/master/card-drag.html)
(admitedly it's dragging in the wrong direction, oops). I found that [GSAP
TweenMax](http://greensock.com/tweenmax), which comes recommended by Google in
the aforementioned guide, was extremely useful in building complex animations
using timelines.

As it often happens for me, the topic that was interesting me greatly in my
spare time serentipitously became useful for my work, as we took on a
short-term contract to build out a drag-and-drop interface to build up those
famous family member stick-figure stickers for rear car windows. So I first
[implemented the app to spec](http://stickers.1000familiesbc.com/), then took
it upon myself to build an [unofficial mobile responsive version using TweenMax
and the Draggable module](http://stickers.1000familiesbc.com/m).  It was one of
the more entertaining projects I've worked on recently, albeit quite small and
simple, because I was able to explore a topic that I was very interested in at
the time.

In doing this project I tried to use `angular-animate` with Angular 1.2 (needed
to support IE8+), but ended up frustrated and ditched it for TweenMax.

Just a couple of weeks ago, learning [Famo.us](http://famo.us) became an itch
that I just had to scratch when I stumbled upon the [Famo.us Angular by Zack
Brown](https://www.youtube.com/watch?v=irbm9MCnznw) presentation at ng-europe,
while catching up on new Angular features. The [demo
site](http://famo.us/integrations/angular) was just too impressive to ignore.
I highly recommend checking it out and going through [Famo.us
University](http://famo.us/university/home/).  Famo.us, in a nutshell,
abstracts the DOM and Canvas and performs all animations and updates matrix3d
property to animate elements, based off of values calculated in their own
rendering engine that runs in JavaScript. It's a bit like React.js and it's use
of a virtual DOM to minimize actual DOM interactions. Famo.us also provides a
3d physics engine to give natural, native feeling effects. It's pretty badass
and I highly recommend checking it out. I modified the Famo.us University
slideshow tutorial example to get data from my Instagram feed, [check it out
here on your desktop or mobile
device](http://brentvatne.ca/famo.us-practice/slideshow-instagram/) - this is
not particularly complicated to implement.

But I continued to work my way through the ng-europe presentations and was very
impressed by Matias Niemelä's (of [yearofmoo](yearofmoo.com) fame) [presentation
on animations in Angular 1.3](https://www.youtube.com/watch?v=3hktBbxFxSM).

--------------------------------------------------------------------------------

Below is just a recap of what I learned from that presentation. Be sure to use
Angular 1.3! It includes many improvements over animations in 1.2.

### Getting started
You'll need to add `angular-animate.js` to your site, then add `ngAnimate` as a
module dependency to your app.

### With CSS
To make a generic `zoom` aniation, we can just define the CSS properties of
elements with the `zoom` class at various states in the animation cycle.


  {% highlight css %}
// This is where we start from: we go from not being 'visible' (eg: ng-if
 // evaluates to false) to 'visible' (eg: ng-if evaluates to true)
 //
 .zoom.ng-enter {
   transition: 0.5s linear all;
   position: relative;
   left: -200px;
   opacity: 0;
 }
 
 // This is the state that it will be in when it is 'visible' - (eg: ng-if
 // evaluates to true)
 //
 .zoom.ng-enter-active {
   left: 0;
   opacity: 1;
 }
  {% endhighlight %}

### With JavaScript
We can do the same with JavaScript by registering an animation for the class
using `app.animation('.zoom')`

  {% highlight javascript %}
app.animation('.zoom', function() {
   return {
     enter: function(element, done) {
       // Set up the initial state, like ng-enter above
       element.css({position: relative; left: '-200px', opacity: 0});
 
       // Here we animate to the visible state, just like ng-enter-active above,
       // and then we fire the done callback that was passed in.
       element.animate({left: '0px', opacity: 1}, done);
     }
   }
 });
  {% endhighlight %}

The above example uses jQuery to animate, but you could use anything as long as it
calls the done callback when the animation is complete. As I said in the preface above,
I recommend [GSAP TweenMax](http://greensock.com/tweenmax).

Events that can be hooked into are dependent on the directive, but a comprehensive list for Angular 1.3 is:
`enter` `leave` `move` `addClass` `removeClass`.
[See which work with which directives here](http://url.brentvatne.ca/17TcB)

### Going back to CSS

You can add a stagger to the enter and leave animations on `ng-repeat` (some
period of time in between when an animation starts on an item and its following
item), just add `-stagger` to the end of the `ng-enter` and `ng-leave` classes.

  {% highlight css %}
.zoom.ng-enter-stagger, .zoom.ng-leave-stagger {
   transition-delay: 0.2s;
   transition-duration: 0s;
 }
  {% endhighlight %}

### ng-animate-children

Run animations on children, but all at the same time. You sequence animations
by adding delays, but that adds a lot of coupling and is just plain messy. So
`ngAnimateLayout` was created to solve this.

### ng-animate-layout

This module is still experimental. The current API allows us to define
sequences as follows:

  {% highlight html %}
<div ng-if="visible" class="dark-stage">
   <ng-animation>
     <ng-animate-sequence on="enter">
       <ng-animate selector="li" stagger="500" apply-classes="zoom"></ng-animate>
     </ng-animate-sequence>
   </ng-animation>
 
   <ul>
     <li>..</li>
     <li>..</li>
     <li>..</li>
   </ul>
 </div>
  {% endhighlight %}

This is really neat because it allows us to animate sequences in the Angular way - declaratively, through markup. The impetus for this sequencer was Material
Design - in particular the [Hierarchical timing](http://www.google.com/design/spec/animation/meaningful-transitions.html#meaningful-transitions-visual-continuity)
principle.

### ng-animate-keep

Okay that is pretty cool, but I believe `ng-animate-keep` is what makes `ng-animate-layout` truly innovative and useful.
Inspired by the Material Design principle [Visual continuity](http://www.google.com/design/spec/animation/meaningful-transitions.html#meaningful-transitions-visual-continuity),
it allows you to persist an element across transitions without having to manually create a copy of the element and manipulate it.

Check out the [example from the yearofmoo ng-europe presentation](https://www.youtube.com/watch?v=3hktBbxFxSM#t=1310).


  {% highlight html %}
// Assumes that this is within some view and selectItem() triggers a view change
 <div class="dark-stage">
   <ng-animation>
     <ng-animate-sequence on="leave">
       <ng-animate-keep selector=".photo" stagger="500">
       <ng-animate selector="li" stagger="500" apply-classes="zoom"></ng-animate>
     </ng-animate-sequence>
   </ng-animation>
 
   <ul>
     <li ng-repeat="item in items">
       <img src="xxx.png" class="photo" ng-click="selectItem(item)">
     </li>
   </ul>
 </div>
  {% endhighlight %}

`ng-animate-keep` will "look for elements to persist across views, and then match the element in the second view".

### $animate promises

Rather than using callbacks, you can use the standard Angular way of callbacks.
Warning: this promise does not run within the digest cycle, so you need to use
$scope.$apply if you're going to change some $scope value when the animation is
completed.

### $animate.animate

Inline animations - pass in values for the animation:

  {% highlight javascript %}
$animate.animate(angular.element(box),
   // From state
   {},
   // To state
   {left: coords[0], top: coords[1]},
   // Class to add
   'active'
 );
  {% endhighlight %}

Assuming that the `box` element has some CSS applied for transitioning
(eg: `transition: all 0.2s linear;`) - or we can animate using the
`app.animation` function:

  {% highlight javascript %}
app.animation('.zoom', {
   return {
     // Exactly the same for removeClass
     addClass: function(element, className, done, styles) {
       // styles contains the inline styles, injected from $animate.animte
       console.log(styles.from);
       console.log(styles.to);
     }
   }
 })
  {% endhighlight %}

### Read more
- [Staggering Animations in AngularJS](http://www.yearofmoo.com/2013/12/staggering-animations-in-angularjs.html)
- [Remastered Animation in AngularJS 1.2](http://www.yearofmoo.com/2013/08/remastered-animation-in-angularjs-1-2.html) - this one is a bit out of date, but still good reading as long as you watch the presentation about ngAnimate in 1.3 so you know what has changed.
