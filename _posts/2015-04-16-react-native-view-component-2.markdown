---
layout: post
title: React Native View Components (2/2)
class: react-native-view-components-2
description: Fixing a bug in SliderIOS by subclassing UISlider
---

[Part 1](link-here) discussed how to wrap existing iOS view components
and pass the attributes from JavaScript. This works great for simple use
cases like wrapping `UISlider`, but you will quickly run into situations
where you need to exercise a bit more control over your components on
the Objective-C side. We don't have to look far to find an example of
this.

At the time of writing, the `SliderIOS` component has a bug where
the `value` is always clamped between `0.0` and `1.0`; even if we set
the `maximumValue` to `100.0`:

{% highlight javascript %}
var React = require('react-native');
var { SliderIOS, } = React;

var MySliderApp = React.createClass({
  render() {
    return <SliderIOS value={75.0} maximumValue={100.0} style={{marginTop: 50}} />
  }
});
{% endhighlight %}

![broken-example](http://brentvatne.ca/images/view-components-2/1-broken.png)

*If you would like to follow along, create a new project with
`react-native init` and put the `SliderIOS` component above into it.*

This occurs because we have no way of indicating in which order our attributes
should be applied, and `UISlider` will ignore values that are outside of
its current `minimumValue` and `maximumValue` range, which defaults to
`0.0` and `1.0` respectively, as you may have guessed. So how do we fix
this? We subclass `UISlider` and override our setters to work regardless
of their application order.

Let's create `RCTSlider.h` and put it inside of
`node_modules/react-native/React/Views` (the same directory as
`RCTSliderManager`. It should look like this:


{% highlight objc %}
#import <UIKit/UIKit.h>

@interface RCTSlider : UISlider

@end
{% endhighlight %}

Simple enough, we define `RCTSlider` as a subclass of `UISlider` and in
order to do that we import `UIKit`. Next we create `RCTSlider.m`:

{% highlight objc %}
#import "RCTSlider.h"

@implementation RCTSlider {
  float _value;
}

- (void)setValue:(float)value
{
  _value = value;
  [super setValue:value];
}

@end
{% endhighlight %}

We create an instance variable `_value`, a float where we will track the
what the slider `value` property is set to irrespective of the range
constraints that would normally clamp it. Every time `setValue:value` is
called, we store the given value in `_value` and then forward the call
to the superclass to handle it as it did before.

With this in place, we can override `setMinimumValue:minimumValue` and
`setMaximumValue:maximumValue` to ensure that `value` is re-applied
afterwards, in case it was improperly clamped due to `setValue:value`
being applied before the range was initialized. Add the following code
before `@end` in `RCTSlider.m`

{% highlight objc %}
- (void)setMinimumValue:(float)minimumValue
{
  [super setMinimumValue:minimumValue];
  [super setValue:_value];
}

- (void)setMaximumValue:(float)maximumValue
{
  [super setMaximumValue:maximumValue];
  [super setValue:_value];
}
{% endhighlight %}

The last step is to wire `RCTSlider` up with the `RCTSliderManager`
class that we looked at in part 1:

{% highlight objc %}
- (UIView *)view
{
  RCTSlider *slider = [[RCTSlider alloc] init]; // This line is all that changes!
  [slider addTarget:self action:@selector(sliderValueChanged:) forControlEvents:UIControlEventValueChanged];
  [slider addTarget:self action:@selector(sliderTouchEnd:) forControlEvents:UIControlEventTouchUpInside];
  return slider;
}
{% endhighlight %}

All we needed to do was replace `UISlider` in `UISlider *slider =
[[UISlider alloc] init];` with `RCTSlider`, and now `RCTSliderManager`
will instantiate our `RCTSlider` class instead when the `<SliderIOS />`
component is used.

Lastly, we need to ensure that we add these new files to `Compile
Sources`:

![compile-sources](http://brentvatne.ca/images/view-components-2/2-compile-sources.gif)

And if we run our broken app, we should see this now:

![fixed-example](http://brentvatne.ca/images/view-components-2/3-fixed.png)


Those are the basics of creating an Objective-C backed React Native view
component. There's a lot more that you can do - I recommend reading
through some of the standard library and community components such as
[react-native-video](https://github.com/brentvatne/react-native-video)
to see what else can be done, and ping me at
[@notbrent](https://twitter.com/notbrent) on Twitter or IRC if you get
stuck on anything. You might also want to check out
[Packaging a React Native Component](http://brentvatne.ca/packaging-react-native-component),
where I describe how you can ship your Objective-C backed components
through npm.

More comprehensive documentation is upcoming in the React Native docs,
give me a week or two to put it together. Have fun!
