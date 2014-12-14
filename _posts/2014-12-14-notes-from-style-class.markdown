---
layout: post
title: Style & Class Conference 2014
class: style-class-2014
description: My notes from the conference that took place in Vancouver on December 13, 2014.
---

# Val Head - All The Right Moves
- Most designers/developers not trained in motion design
- Keep interface animations flexible
  - eg: stripe remember me checkbox, if you reverse in the middle it should
        just go back
  - it should feel like a conversation - clicks and taps should be listened to
  - be open for input at any time
  - keyframe animations vs transitions - keyframe easier to fall into unresponsiveness
- Prototype all ideas with animations
- Speed/timing
  - .2s / .6s are good for 'small' interactions
- ease-out feels more responsive typically
  - starts fast, ends slow
  - lots of variations
- complex easing needs more time to be readable
- match motion to your message
  - eg: dots game, bouncy motion in the menu, similar to the game
- think stage, not page
  - often have a 'slideshow' mentality on the web - multiple things
    coming in together then going out
  - eg fontshop carousel boxes disappearing and re-appearing
  - leave things on screen that will stay on screen
- animation is a design tool
- http://valhead.com/ui-animation for more resources

# Alaine Mackenzine - Content strategy for designers
- Kristina Halvorson
- practice of planning, creation, delivery and governance
- Tools include the following:
- Content audit
  - Find out what there is now, and if it is good
- Voice and tone
  - Define your client's personality, understand your user's emotions,
    ensure consistency
  - http://voiceandtone.com/
  - "this-but-not-that" list
    - Fun but not childish, clever but not silly, confident but not
      cocky, smart but not stodgy, cool but not alienating
- Content models
  - elements, priorities and relationships
  - list all of the elements, prioritize them
    - may be a conflict between what org and users think is important
  - write numbers on different sections of content in a mockup to
    indicate priority
- Governance plan

# Vitaly Friedman - Improving smashing mag's performance
- No diference between being down or too slow
- "Core HTML and CSS"
  - Priority list for content and styles to define "core" in The
    Guardian re-design
  - Core content, Enhancement (JS, Geolocation, touch, enhanced css, web
    fonts, widgets), Leftovers (analytics, advertising, third-party
content) in
- Load core content first, then enhancement on DOMContentReady, then
  Leftovers on load
  -  Load JS with async and defer
  - Split browsers into two groups: HTML4, HTML5
    - querySelector, localStorage, addEventlistener: smart browser
    - otherwise, dumb browser
    - this can replace modernizr
- Fonts downloaded? -> Download base64 encoded in json, cache in
  localstorage, then show fonts
- http://www.theguardian.com/uk?view=mobile - better performing on
  mobile than desktop even
- Key changes:
- Load critical css inline
- CSS on load
- Store web fonts in localStorage + cookies
- Defer advertising, tracking, and non-critical
- srcset and picture element
- SPDY - 64% faster page loads
- Improving smashing magazine's performance: a case study - article
  summarizing this talk

# Meredith - Ladies learning code
- Socrata, Code & Coffee
- Lots of events coming up in the new year in Vancouver, worth checking
  out perhaps to volunteer or suggest to friends.

# Chelsea Klukas - Human-Centered Design
- makeFashion
- Lululemon sweater with hair elastic
- "Beautiful is playful and disarming"
- What if David Bowie designed airlines or Lady Gaga Excel?

# Accessibility and CSS
- Screen readers - read out in very fast voice
- High contrast - black background, white text
- Voice recognition - link based on text, or grid if can't be triggered
  with it, broken down into 1-9 recursively until selecting - "coarse"
pointer, eg: finger. Fine pointer accurate to pixel, eg: mouse.
- Trackball - small to hit small target
- Keyboard - some users only use it, incapable of hovering
- Try it: Jaws / ChromeVox
- http://wave.webaim.org/ evaluate it, check out the guidelines
- Colorblindness - 10% of men are colorblind
  - Don't depend on just color to indicate states!
  - Icons
- Never remove keyboard focus styles - just design better ones if you
  don't like - reuse hover styles even
- Rather than "Details", for example, have the button use the server
  name - multiple buttons with the same name.
  - speak: none
  - Never put important content in css content attributes
  - Increase hit area for buttons - the bigger it is, the faster to
    interact with
- Filament group: bulletproof accessible icon fonts
- aria-hidden has much wider support than speak: none
- Offscreen class: pos abs, heigth 1px, width: 1px, overflow: hidden,
  clip rect 1px 1px 1px 1px (by Snook) - because screenreaders don't
  read display: none. Bootstrap has 'assistive text'
- "Don't sell it, do it"
  - Once you learn the basics, it integrates into workflow without it
    taking extra time.
  - Makes it better for everyone
- http://stephaniehobson.ca/

# John Allsopp - Well, how did we get here?
- We have a chance to shape the medium of the web.
