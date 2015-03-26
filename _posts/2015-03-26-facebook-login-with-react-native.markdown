---
layout: post
title: Native Facebook login with React Native
class: facebook-login-react-native
description: RCTBridgeModule and NativeModules make this easy!
---

React-native provides a
[RCTBridgeModule/NativeModules](http://facebook.github.io/react-native/docs/nativemodulesios.html)
API which allows you to write Objective C to directly access native APIs when necessary.

One case where this is particularly useful is to take advantage of the native authentication libraries,
such as sign in through Facebook. I'll explain how to do that here.

If you don't have a project you're working on already, create one with `react-native init FacebookLogin`
and then go to [https://developers.facebook.com/](https://developers.facebook.com/) and create a new
app. Follow the "Getting Started" guide for iOS, and make sure that you do the "Track App Installs and App Opens" step -
it looks like it might be unnecessary but login requests will always return as cancelled if this is skipped. Another
thing to keep in mind for that step is that we will already have an `application didFinishLaunchingWithOptions`
function in our `AppDelegate` - you can just replace the `return YES` at the end of the existing one with
the return statement in the guide.

Once this is in place, replace the index js file with:

{% highlight javascript %}
var React = require('react-native');
var {
  AppRegistry,
  StyleSheet,
  Text,
  View,
  TouchableHighlight
} = React;

var FacebookLoginManager = require('NativeModules').FacebookLoginManager;

var FacebookLogin = React.createClass({
  getInitialState() {
    return {
      result: '...'
    }
  },

  componentDidMount() {
    var self = this;
  },

  login() {
    FacebookLoginManager.newSession((error, info) => {
      if (error) {
        this.setState({result: error});
      } else {
        this.setState({result: info});
      }
    });
  },

  render() {
    return (
      <View style={styles.container}>
        <TouchableHighlight onPress={this.login}>
          <Text style={styles.welcome}>
            Facebook Login
          </Text>
        </TouchableHighlight>
        <Text style={styles.instructions}>
          {this.state.result}
        </Text>
      </View>
    );
  }
});
{% end %}

In XCode, create FacebookLoginManager.h and paste this in:

{% highlight objective-c %}
#import "RCTBridgeModule.h"

@interface FacebookLoginManager : NSObject <RCTBridgeModule>
@end
{% end %}

Also create a FacebookLoginManager.m and paste this in:

{% highlight objective-c %}
#import "FacebookLoginManager.h"
#import "FBSDKCoreKit/FBSDKCoreKit.h"
#import "FBSDKLoginKit/FBSDKLoginKit.h"

@implementation FacebookLoginManager

- (void)newSession:(RCTResponseSenderBlock)callback {
    RCT_EXPORT();

    FBSDKLoginManager *login = [[FBSDKLoginManager alloc] init];
    [login logInWithReadPermissions:@[@"public_profile", @"email"] handler:^(FBSDKLoginManagerLoginResult *result, NSError *error) {
        
        if (error) {
            callback(@[@"Error", [NSNull null]]);
        } else if (result.isCancelled) {
            callback(@[@"Canceled", [NSNull null]]);
        } else {
            FBSDKAccessToken *token = result.token;
            NSString *tokenString = token.tokenString;
            NSString *userId = token.userID;
            NSDictionary *credentials = @{ @"token" : tokenString, @"userId" : userId };
            callback(@[[NSNull null], credentials]);
        }
    }];
}

@end
{% end %}

When you run the app and tap on the Facebook Login text, you should be prompted
to approve permissions for Facebook - once that is complete, you will see the token and user id
output onto the screen. You can use this data with the Facebook JavaScript SDK to fetch other
information, or pass it along to a server to initialize an account - it's up to you!