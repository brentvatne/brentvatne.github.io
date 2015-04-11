---
layout: post
title: React Native View Components: Part 1
class: react-native-view-components-1
description: Explores the SliderIOS implementation to understand basic component wrapping techniques.
---

React Native has been open source for about two weeks now, so it's
incredible that it's already possible to build functional and useful
apps (for example, 2048 app, HackerNews reader) without even leaving the
JavaScript world. But for most non-trivial apps, you'll quickly find
yourself running into issues such as lack of support for `multiline` on
`<TextInput>`, accessing an unwrapped API such as the Google Login SDK,
needing to display a gradient, presenting a modal window, playing and
controlling the playback options of a video or audio file, adding a blur
effect and so on. The community is quickly building components to solve
these problems but you want to use react-native now.

This is where it becomes useful to know how to use the excellent bridge
API. In my [previous post](http://brentvatne.ca/facebook-login-with-react-native/) I
discussed how to use the bridge to access an API without that is
unrelated to the views, focusing on the Facebook login SDK. Now I'll
talk about wrapping components that are actually displayed on the
screen.

The minimum required to create our own native component is a
`RCTViewManager` class, and a JavaScript wrapper class that references
it. In this first part, I'll be discussing how to create this type of
component.  In part two, I'll look at wrapping `RCTView` as well to gain
more control, and managing it with your `RCTViewManager`.  I encourage
you to read these even if you don't know Objective-C (my knowledge of it
is about two weeks old), it's not that hard to pick up what you need to
know to build your own components.

### [RCTSliderManager.h](https://github.com/facebook/react-native/tree/0.3.11/React/Views/RCTSliderManager.h)

{% highlight objc %}
#import "RCTViewManager.h"

@interface RCTSliderManager : RCTViewManager

@end
{% endhighlight %}

Our view manager needs to inherit from `RCTViewManager` and must follow
the naming convention ComponentNameManager. You can add your own prefix
if you like, such as `BV` (my initials) or `RN`, but you should avoid
using `RCT` unless your intention is to have the library merged into
`react-native`. That's all of the header file this time.

### [RCTSliderManager.m](https://github.com/facebook/react-native/tree/0.3.11/React/Views/RCTSliderManager.m)

{% highlight objc %}
#import "RCTSliderManager.h"

#import "RCTBridge.h"
#import "RCTEventDispatcher.h"
#import "UIView+React.h"
{% endhighlight %}

You'll always include your Manager's header file, which is done on the
first line. `RCTBridge.h` should be included any time you're dealing
with events and passing them throughout the app, as should
`RCTEventDispatcher.h`. You'll see compiler warnings/errors in XCode if
you don't do this. If you're using any of the React specific `UIView`
interface, such as `reactTag`, you will need to include `UIView+React.h`
- don't get too hung up on this though, XCode warnings and reviewing
other files make this hard to mess up. _Warning: make sure you don't import
the `.m` files, you will almost certainly face a bunch of cryptic errors._

{% highlight objc %}
@implementation RCTSliderManager

RCT_EXPORT_MODULE()

- (UIView *)view
{
  UISlider *slider = [[UISlider alloc] init];
  [slider addTarget:self action:@selector(sliderValueChanged:) forControlEvents:UIControlEventValueChanged];
  [slider addTarget:self action:@selector(sliderTouchEnd:) forControlEvents:UIControlEventTouchUpInside];
  return slider;
}
{% endhighlight %}

In our implementation of a View Manager we always have to call the
`RCT_EXPORT_MODULE()` macro to make the class available to JavaScript,
and we must also define the `- (UIView *)view` method that creates the
component that the manager is responsible for and returns it. In this
case, it also registers callbacks to the view manager class when the
slier is changed or touch ends.

{% highlight objc %}
- (void)sliderValueChanged:(UISlider *)sender
{
  NSDictionary *event = @{
    @"target": sender.reactTag,
    @"value": @(sender.value),
    @"continuous": @YES,
  };

  [self.bridge.eventDispatcher sendInputEventWithName:@"topChange" body:event];
}

- (void)sliderTouchEnd:(UISlider *)sender
{
  NSDictionary *event = @{
    @"target": sender.reactTag,
    @"value": @(sender.value),
    @"continuous": @NO,
  };

  [self.bridge.eventDispatcher sendInputEventWithName:@"topChange" body:event];
}
{% endhighlight %}

These are the callback methods that we supplied for the slider in the
initializer. We take the slider instance and build an event dictionary
that includes, at the bare minimum, the `target` component, and then any other
information that you would like to provide. We then pass that along
across the bridge using the event dispatcher method `sendInputEventWithName`,
referencing the "topChange" event [which maps to
onChange](https://github.com/facebook/react-native/tree/0.3.11/React/Modules/RCTUIManager.m#L1149-L1154)
in the UIManager. This means that our interface to this in JavaScript
will be through the `onChange` prop in our wrapper, which we will see in
a moment.

{% highlight objc %}
RCT_EXPORT_VIEW_PROPERTY(value, float);
RCT_EXPORT_VIEW_PROPERTY(minimumValue, float);
RCT_EXPORT_VIEW_PROPERTY(maximumValue, float);
{% endhighlight %}

At the bottom of `RCTSliderManager.m` we tell the manager that it
accepts `value`, `minimumValue`, and `maximumValue` props which [map
exactly to the properties on
UISlider](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UISlider_Class/),
and will automatically convert the JavaScript values we pass over the
bridge to the `float` type before setting them on the `UISlider`
instance that we returned from the initialization `- (UIView *)view`
method. Often it won't make sense to make our React component prop names
directly to the iOS prop names, and we'll look at how to deal with that
in the next article.

This is all we need for the Objective-C side of things, now we move over
to JavaScript.

### [SliderIOS.js](https://github.com/facebook/react-native/blob/v0.3.11/Libraries/Components/SliderIOS/SliderIOS.js)

Let's read this file bottom up and skip the `require`s entirely.

{% highlight javascript %}
var validAttributes = {
  ...ReactIOSViewAttributes.UIView,
  value: true,
  minimumValue: true,
  maximumValue: true,
};

var RCTSlider = createReactIOSNativeComponentClass({
  validAttributes: validAttributes,
  uiViewClassName: 'RCTSlider',
});
{% endhighlight %}

[ReactIOSViewAttributes](https://github.com/facebook/react-native/blob/v0.3.11/Libraries/ReactIOS/ReactIOSViewAttributes.js)
is a module that we require which exposes property objects for `UIView`
and `RCTView`, we need to merge the attributes that we want to use with
our component in. For simple data types (numbers, booleans, strings) the
corresponding value is always `true`, and with complex data types
(arrays, objects) we can specify a differ function instead of `true`:

{% highlight javascript %}
// This isn't done in SliderIOS, it's an example
var cakeDiffer = function(cakeA, cakeB) {
  return cakeA.superHeroName != cakeB.superHeroName;
}

var validAttributes = {
  ...ReactIOSViewAttributes.UIView,
  birthdayCakeDesigns: {diff: cakeDiffer},
  customerName: true,
};
{% endhighlight %}

If you don't provide a differ function, the default is
[deepDiffer](https://github.com/facebook/react-native/blob/v0.3.11/Libraries/Utilities/differ/deepDiffer.js).

Next, we call the
[createReactIOSNativeComponentClass](https://github.com/facebook/react-native/blob/72d3d724a3a0c6bc46981efd0dad8f7f61121a47/Libraries/ReactIOS/createReactIOSNativeComponentClass.js)
function giving it the `validAttributes` object and the
`uiViewClassName` corresponding to the Manager that we created
previously, but leaving out the `Manager` suffix - it's a convention,
therefore React can do that typing for us.

Beautiful. If we exported only this, our component would work, let's
pretend that it had `module.exports = RCTSlider` at the end, then after
we require it in our view we can use it like this:

{% highlight javascript %}
logValue(newSliderValue) {
  console.log(newSliderValue);
},
render() {
  return (
    <RCTSlider style={styles.slider}
               value={50}
               minimumValue={1}
               minimumValue={100}
               onChange={this.logValue} />
  );
},
{% endhighlight %}

But you will find much more than this in `SliderIOS.js`, because it's
good manners to wrap the native component in React class that validates
`propTypes`, applies any default styles, and in some cases (this one
included) wraps an event callback.

{% highlight javascript %}
var SliderIOS = React.createClass({
  mixins: [NativeMethodsMixin],

  propTypes: {
    style: View.propTypes.style,
    // Other props omitted, look at the source if you like
    onValueChange: PropTypes.func,
    onSlidingComplete: PropTypes.func,
  },

  _onValueChange: function(event: Event) {
    this.props.onChange && this.props.onChange(event);
    if (event.nativeEvent.continuous) {
      this.props.onValueChange &&
        this.props.onValueChange(event.nativeEvent.value);
    } else {
      this.props.onSlidingComplete && event.nativeEvent.value !== undefined &&
        this.props.onSlidingComplete(event.nativeEvent.value);
    }
  },

  render: function() {
    return (
      <RCTSlider
        style={[styles.slider, this.props.style]}
        value={this.props.value}
        maximumValue={this.props.maximumValue}
        minimumValue={this.props.minimumValue}
        onChange={this._onValueChange}
      />
    );
  }
});
{% endhighlight %}

So we create our own React component and mixin
[NativeMethodsMixin](https://github.com/facebook/react-native/blob/master/Libraries/ReactIOS/NativeMethodsMixin.js),
which adds some important functions such as `setNativeProps`, `measure`
and `measureLayout`. It's boilerplate.

`View.propTypes.style`
([ViewStylePropTypes](https://github.com/facebook/react-native/blob/master/Libraries/Components/View/ViewStylePropTypes.js))
defines the expected structure of a style prop - if your component has
custom style properties, you'll want to `merge` them into this.

The `onSlidingComplete` and `onValueChange` props... wait, I said
`onChange` was the prop that we would use to get the updated value,
what's going on here? Well, if you jump down the `render` function you
will see that it passes in `this._onValueChange` into the `onChange`
prop for `RCTSlider`, and if you follow that you will see what's going
on. A few things to notice:

- This function reads the raw event via `event.nativeEvent`, which you
  will recognize has the `value` and `continous` properties that we set
on the dictionary in Objective C.
- Based on the `event.nativeEvent` data, it fires one of the different
  passed in callbacks.

So the `SliderIOS` wraps the `RCTSlider` component in order to valiate
props, apply base styles, and split up our `onChange` callback into
`onSlidingComplete` and `onValueChange`, depending on the data passed
back from the native event.

### Adding to it

Given what we know now, think about how you might add support for the
`UISlider` properties `minimumTrackTintColor` and
`maximumTrackTintColor`.  Try to implement this yourself - create a new
project, open up the `node_modules/react-native` directory and find the
files we discussed above.  Note that when you make changes to Objective
C code, you will need to re-compile and run the simulator again.

Done? [Check out my solution](https://github.com/facebook/react-native/pull/799/files).

[Check out some of my react-native
libraries Github](https://github.com/brentvatne), read the code, explore
the React Native source and you won't even need to wait for part two of
this post. Thanks for reading!
