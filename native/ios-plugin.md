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

