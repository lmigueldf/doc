# Developing Reusable Native Plugins

The Game Closure DevKit supports a plugin system that enables you to develop unique features for your game in native code and JavaScript SDK that will seamlessly interoperate with and persist through upgrades to new versions.

These plugins can be easily shared privately with your internal development process or shared back to Game Closure to benefit the rest of the HTML5 game development community.

Plugins allow your game to use a minimum set of permissions, which will prevent users from getting scared off.

Native plugin code has access to the `manifest.json` file contents so you can configure the plugins through that file.

Native code and JavaScript interact through a message passing interface.  The JS Event System is used to send events to JavaScript, and a NATIVE function is used to send events to native code.  This native interface is the same for iOS and Android.

The plugin system for iOS consists of several pieces:  There will be a JavaScript wrapper for the native functions.  A configuration file describes what files and dependencies need to be added to the native project before building.  And of course the native source code that interacts with your JavaScript wrapper.

## JavaScript

The JS wrapper code can be specialized for each target.  iOS, Android, and browser builds can all use different versions of your JS wrapper.

## Native Plugin Example: GeoLocation

To demonstrate the plugin system, take the [GeoLocation Plugin](https://github.com/gameclosure/geoloc) as an example.

### Using plugins from your game

The user of the plugin must include it in the `manifest.json` file under the "addons" section:

~~~
	"addons": [
		"geoloc"
	],
~~~

Furthermore at the top of the game's `src/Application.js`:

~~~
import geoloc;
~~~

This imports the `geoloc.js` file under `devkit/addons/geoloc/js/`.

There are actually two versions of this file for GeoLocation.

The `js/native/geoloc.js` file is used for native iOS or Android builds.  And the `js/geoloc.js` file is used for web builds.  This distinction is important since web browsers already support geolocation, but on native platforms JavaScript code must be included to interact with the native code.

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

### Android Plugin: android/config.json

Under `addons/geoloc/android/config.json` is the configuration used while building Android targets with the GeoLocation plugin.

It specifies Java code, libraries, JARs, and XML required to build the Android plugin:

~~~
{
		"copyFiles": [
			"GeolocPlugin.java"
		],
		"libraries": [
		],
		"jars": [
		],
		"injectionXML": "manifest.xml",
		"injectionXSL": "manifest.xsl"
}
~~~

##### copyFiles

Java source code files in the `addons/geoloc/android/` directory that will be incorporated into the Android build.  The `basil` build scripts will pull the package name out of the Java files.

The TeaLeaf Java source code will be built first, and then the Activity for your game will be built.  It is in this second build step where your plugin code is built.

If your package is `com.tealeaf.plugin.plugins`, then the file will be copied into the destination build directory under `src/com/tealeaf/plugin/plugins` and the `ant` build will incorporate them.

##### libraries

A list of .a library files in the `addons/geoloc/android/` directory.

##### jars

A list of .jar files in the `addons/geoloc/android/` directory.

##### injectionXML

To add permissions and other AndroidManifest.xml keys, you can edit the `manifest.xml`.

For example to add permissions for the GeoLocation plugin, the ACCESS_FINE_LOCATION permission was added:

~~~
	<!--START_PLUGINS_MANIFEST-->
		<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
	<!--END_PLUGINS_MANIFEST-->
	<!--START_PLUGINS_APPLICATION-->
	<!--END_PLUGINS_APPLICATION-->
~~~

##### injectionXSL

An advanced feature, if you want to use XSLT parsing to modify the AndroidManifest.xml then you can provide a `manifest.xsl` that get passed through xslt during the build.

### Android Plugin: Code

Your plugin class must be packaged under `com.tealeaf.plugin.plugins` with a `package com.tealeaf.plugin.plugins;` line:

~~~
package com.tealeaf.plugin.plugins;
~~~

Your Java code will implement the IPlugin interface that is imported with a `import com.tealeaf.plugin.IPlugin;` line.  There are a number of other useful imports that are worth looking into:

~~~
import com.tealeaf.plugin.IPlugin;
import com.tealeaf.event.*;
import com.tealeaf.EventQueue;
import com.tealeaf.logger;
import org.json.JSONObject;
import org.json.JSONArray;
import org.json.JSONException;
~~~

The IPlugin interface provides a number of callbacks for common events that occur while running games, which you can react to:

~~~
public interface IPlugin {
	public void onCreate(Activity activity, Bundle savedInstanceState);
	public void onCreateApplication(Context applicationContext);
	public void onResume();
	public void onStart();
	public void onPause();
	public void onStop();
	public void onDestroy();
	public void onNewIntent(Intent intent);
	public void onActivityResult(Integer request, Integer result, Intent data);
	public void setInstallReferrer(String referrer);
}
~~~

##### Handling JavaScript Events

For the Android plugin system it is important that the JavaScript code specifies the correct plugin class name and method to call.  Note that the above class matches the JavaScript name:

~~~  
NATIVE.plugins.sendEvent("GeolocPlugin", "onRequest", '{"method":"getPosition"}');
~~~

So the corresponding class for this would expose an onRequest function:

~~~
public class GeolocPlugin implements IPlugin {
	public void onRequest(String jsonData) {
		try {
			JSONObject data = new JSONObject(jsonData);

			String method = data.optString("method", "getPosition");

			// Handle getPosition()
			if (method.equals("getPosition")) {
				logger.debug("{geoloc} Got position request");
			}
		} catch (Exception e) {
			logger.log(e);
		}
	}
}
~~~

##### Sending JavaScript Events

To set up an Event that will get passed back to JavaScript write code that looks like this:

~~~
public class GeolocEvent extends com.tealeaf.event.Event {
	boolean failed;
	double longitude, latitude;
	public GeolocEvent(double longitude, double latitude) {
		super("geoloc");
		this.failed = false;
		this.longitude = longitude;
		this.latitude = latitude;
	}
}
~~~

To pass the event to JavaScript, create a new one and push it into the `EventQueue`:

~~~
EventQueue.pushEvent(new GeolocEvent(83.2106, -10.1984);
~~~

And the JavaScript wrapper will listen for this event by registering for it:

~~~
NATIVE.events.registerHandler('geoloc', function(e) {
	if (!e.failed) {
		logger.log("{geoloc} Got response:", e.longitude, e.latitude);
	}
});
~~~

### iOS Plugin: ios/config.json

Under `addons/geoloc/ios/config.json` is the configuration used while building iOS targets with the GeoLocation plugin.

It specifies the code and frameworks needed to build the iOS plugin:

~~~
{
	"code": [
		"GeoLoc.h",
		"GeoLoc.mm"
	],
	"frameworks": [
		"CoreLocation"
	]
}
~~~

##### Code

The `code` array files are added to the Xcode project during build.

The supported file types are:

+ C/C++ Header Files: ".h", ".hpp"
+ C/C++ Source Files: ".c", ".cpp", ".cc"
+ ObjC/ObjC++ Source Files: ".m", ".mm"

Source files are automatically added to the build files in the Xcode project.  They will be visible under the `platforms` group in the file view in Xcode.

Inside the `code` files you can `#import` any file in the TeaLeaf Xcode project by the base file name without being concerned by the absolute path.  For instance, the `GeoLoc.mm` file can include the header that is added with `#import "GeoLoc.h"`.  It can also access any other existing files in the project.

Furthermore these files are referenced instead of being copied.  The generated Xcode project contains references back to the addon files, and any changes to these files in Xcode will be preserved in the file system between `basil` builds.  This accelerates plugin development since you can iterate quickly to test new code.

##### Frameworks

Any frameworks your code requires can be added to the `frameworks` array.

The supported file types are:

+ System frameworks: No extension.
+ Static libraries: ".a"
+ Custom frameworks: ".framework"

The extension determines the type of linkage desired:

Pass in unqualified names such as `CoreLocation` (not `CoreLocation.framework`) for system frameworks.  Be sure to include the header in your plugin code (`#import <CoreLocation/CoreLocation.h>`).

For static libraries, pass in a file path relative to the `ios` directory containing the `config.json` file for iOS.  As an example you may add a new static library under `addons/geoloc/gcif_reader.a`  And in this case the `ios/config.json` file would contain

~~~
...
	"frameworks": [
		"CoreLocation",
		"../gcif_reader.a"
	]
...
~~~

For custom frameworks, use `MyFramework.framework` with a `.framework` extension.

### iOS Plugin: Code

##### GCPluginProtocol

Your plugin class is required to respond to the following selectors in `PluginManager.h`:

~~~
@protocol GCPluginProtocol
@required
- (void) initializeWithManifest:(NSDictionary *)manifest appDelegate:(TeaLeafAppDelegate *)appDelegate;
- (void) sendEvent:(NSString *)eventName jsonObject:(NSDictionary *)jsonObject;
~~~

And it can optionally respond to these additional selectors for your convenience:

~~~
@optional
- (void) didFailToRegisterForRemoteNotificationsWithError:(NSError *)error application:(UIApplication *)app;
- (void) didReceiveRemoteNotification:(NSDictionary *)userInfo application:(UIApplication *)app;
- (void) didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken application:(UIApplication *)app;
- (void) didReceiveLocalNotification:(UILocalNotification *)notification application:(UIApplication *)app;
- (void) applicationDidBecomeActive:(UIApplication *)app;
- (void) applicationWillTerminate:(UIApplication *)app;
- (void) handleOpenURL:(NSURL *)url;
@end
~~~

If you need additional selectors please discuss them with us.

##### Header

The GeoLocation plugin header file describes an @interface class that derives from GCPlugin.  Any class deriving from the GCPlugin will be instantiated via Objective C reflection when your game starts, so it is important to `#import "PluginManager.h"` so that the GCPlugin declaration is visible to the Xcode compiler at build time and your code can compile.

~~~
#import "PluginManager.h"
#import <CoreLocation/CoreLocation.h>

@interface GeoLocPlugin : GCPlugin<CLLocationManagerDelegate>
@property (nonatomic, retain) CLLocationManager *locationMgr;
@property (nonatomic, retain) CLLocation *lastLocation;
@property (nonatomic, assign) bool requested;
@property (nonatomic, retain) NSDate *lastRequestTimestamp;

- (void)getNewLocation;
@end
~~~

Note also that the GeoLocPlugin derives from GCPlugin and furthermore implements the CLLocationManagerDelegate @protocol so that it can receive GPS updates.

#### Source

Since the header and source files are added as files to the Xcode project, they need to include any headers required as normal.  So the `GeoLoc.mm` file starts by including its header:

~~~
#import "GeoLoc.h"
~~~

Then the @implementation for the @interface begins:

~~~
@implementation GeoLocPlugin

// The plugin must call super dealloc.
- (void) dealloc {
    self.locationMgr = nil;
    self.lastLocation = nil;

	[super dealloc];
}

// The plugin must call super init.
- (id) init {
	self = [super init];
	if (!self) {
		return nil;
	}

	self.locationMgr = [[[CLLocationManager alloc] init] autorelease];
	self.locationMgr.desiredAccuracy = kCLLocationAccuracyBest;
	self.locationMgr.delegate = self;
	self.lastLocation = nil;

	return self;
}
~~~

Your plugin must call `[super init]` on `init` as shown above.  This step is required so that your plugin will receive events from JavaScript.

The plugins should also call `[super dealloc]` on `dealloc` to avoid leaking memory.

The GeoLocation plugin responds to the CLLocationManagerDelegate `locationManager` selector so that it can track GPS position updates.

~~~
- (void) locationManager:(CLLocationManager *)manager
     didUpdateToLocation:(CLLocation *)newLocation
            fromLocation:(CLLocation *)oldLocation {
}
~~~

The GeoLoc plugin also responds to the GCPluginProtocol `initializeWithManifest` selector as required.  It does not have any initialization tasks in this case.

The `manifest` NSDictionary object is an ObjC version of the `manifest.json` file, so you can read any configuration data for your plugin here:

~~~
- (void) initializeWithManifest:(NSDictionary *)manifest appDelegate:(TeaLeafAppDelegate *)appDelegate {
	NSLOG(@"{geoloc} Initialized with manifest");
}
~~~

##### Handling JavaScript Events

To send the event from JavaScript:

~~~
var e = {method:"getPosition"};
NATIVE.plugins.sendEvent("GeolocPlugin", "onRequest", JSON.stringify(e));
~~~

The `sendEvent` selector for the GeoLoc plugin demonstrates how to respond to events from JavaScript:

~~~
- (void) sendEvent:(NSString *)eventName jsonObject:(NSDictionary *)jsonObject {
	if ([eventName isEqualToString:@"onRequest"]) {
		NSString *method = [jsonObject valueForKey:@"method"];

		if ([method isEqualToString:@"getPosition"]) {
			NSLOG(@"{geoloc} Got request");

			[[PluginManager get] dispatchJSEvent:[NSDictionary dictionaryWithObjectsAndKeys:
			@"geoloc",@"name", kCFBooleanTrue, @"failed", nil]];
		}
	}
}

@end
~~~

This code checks that the `eventName` is the expected name, and reads the `method` string from the JSON request object, and if it is equal to "getPosition", then it will respond by dispatching an event back to the JavaScript code.  It uses `[PluginManager get]` to retrieve the current instance of the `PluginManager`.

Note that the first argument to `NATIVE.plugins.sendEvent` on the JavaScript side is ignored.  The second argument is the event name and the third is the JSON object, and these are available to your plugin.  For this reason you should try to pick unique names for your events to avoid conflicts with other plugins.

##### Sending JavaScript Events

The `PluginManger` singleton has a `dispatchJSEvent` selector you can use to send NSDictionary objects back to JavaScript.  The `name` field will be the event name that the JavaScript code must register to listen for.  Any other data in the returned NSDictionary object will also be visible to JavaScript.

In this case the object will look like:

~~~
{
	name: "geoloc",
	failed: true
}
~~~

And the JavaScript wrapper will listen for this event by registering for it:

~~~
NATIVE.events.registerHandler('geoloc', function(evt) {
	if (evt.failed) {
	} else {
	}
});
~~~
