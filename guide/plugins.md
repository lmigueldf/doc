# Developing Reusable Plugins

The Game Closure DevKit supports a plugin system that enables you to develop unique features for your game in native code and JavaScript SDK that will seamlessly interoperate with and persist through upgrades to new versions.

These plugins can be easily shared privately with your internal development process or shared back to Game Closure to benefit the rest of the HTML5 game development community.

Plugins allow your game to use a minimum set of permissions, which will prevent users from getting scared off.

## Android



## iOS

The Plugin system for iOS 

~~~

~~~

## Native Plugin Example: GeoLocation

To demonstrate the plugin system, take the [GeoLocation Plugin](https://github.com/gameclosure/geoloc) as an example.

### Using the GeoLocation plugin from your game

It requires the user of the plugin to include it in the `manifest.json` file under the "addons" section:

~~~
	"addons": [
		"geoloc"
	],
~~~

Furthermore in the `src/Application.js`:

~~~
import geoloc;
~~~

This imports the `geoloc.js` file under `devkit/addons/geoloc/js/`.  There are actually two versions of this file:

The `js/native/geoloc.js` file is used for native iOS or Android builds.  And the `js/geoloc.js` file is used for web builds.  This distinction is important since web browsers already support geolocation, but we need to shim in support for geolocation on native platforms with the plugin.

### Using the Native Plugin Event system from JavaScript

To communicate with your native code from JavaScript, the `NATIVE.plugins.sendEvent` function is used to send data to the plugin, and the `NATIVE.events.registerHandler` function is used to receive data from the plugin.

For example, to send the GeoLocation plugin a request called "onRequest" with a JSON object as a parameter:

~~~
var e = {id:1, method:"getPosition", arguments:{}};
NATIVE.plugins.sendEvent("GeolocPlugin", "onRequest", JSON.stringify(e));
~~~

The plugin will choose a string to use for its event name, and the schema for its events, which will be the same for iOS and Android.

And to receive data from the plugin via the Events subsystem:

~~~
NATIVE.events.registerHandler('geoloc', function(evt) {
  if (evt.failed) {
  } else {
  }
});
~~~

If it is necessary to match a request with the response from the plugin, it is good practice to include an `id` field in the request, which can be provided in the response to match it with the request.  This is not demonstrated here since GeoLocation does not need to match responses with requests.  A JavaScript array keyed with these ids is a good way to remember the callback functions to call when a response arrives.

### iOS Plugin: config.json

Under `addons/geoloc/ios/config.json` is configuration used while building iOS targets with the GeoLocation plugin.

It specifies, a header file, source file, and any necessary frameworks:

~~~
{
	"header": "GeoLoc.h",
	"source": "GeoLoc.mm",
	"frameworks": [
		"System/Library/Frameworks/CoreLocation.framework"
	]
}
~~~

The `header` and `source` file contents are concatenated with any other plugins specified in the game's `manifest.json:"addons"` key and written to `TeaLeaf/platform/PluginManager.h` and `TeaLeaf/platform/PluginManager.mm` during the build.

For this reason it is important to sufficiently namespace your variables to avoid stepping on other plugins' variables.  Writing all of your code as a standard Objective C++ @interface class is sufficient.  More on that later.

Any frameworks or static libraries your code depends on can be added to the `frameworks` array and will be programmatically added to the generated Xcode project at build time.

### iOS Plugin: Code

The plugin

~~~
@interface GeoLocPlugin : GCPlugin<CLLocationManagerDelegate>
@property (nonatomic, retain) CLLocationManager *locationMgr;
@property (nonatomic, retain) CLLocation *lastLocation;
@property (nonatomic, assign) bool requested;
@property (nonatomic, retain) NSDate *lastRequestTimestamp;

- (void)getNewLocation;
@end
~~~
