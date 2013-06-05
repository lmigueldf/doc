# DevKit Native Plugins

The Game Closure DevKit supports a plugin system that enables you to develop unique features for your game in native code and/or JavaScript that will seamlessly interoperate with and persist through upgrades to new versions.

+ These plugins can be easily shared privately with your internal development team or shared back to Game Closure to benefit the rest of the HTML5 game development community.

+ Plugins allow your game to use a minimum set of permissions, which will prevent users from getting scared off.

+ Native plugin code has access to the `manifest.json` file contents so you can configure the plugins through that file.

+ Native code and JavaScript interact through a message passing interface.

The JS Event System is used to send events to JavaScript, and a function is used to deliver events from JavaScript to native code.  This native interface is the same for both iOS and Android.

## Plugin Example: GeoLocation

To demonstrate the plugin system, take the [GeoLocation Plugin](https://github.com/gameclosure/geoloc) as an example.  It will be referenced throughout the documentation.

To install the Geolocation addon, run `basil install geoloc`.  It will be installed under `devkit/addons/geoloc`.  You can also manually install addons under the `addons` directory.

If you would like to add your plugin to the list of plugins that can be automatically installed with `basil install` please open up a discussion with us.

#### Using plugins from your game

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

## Plugin Directory Structure

The plugin system for iOS consists of several pieces:  There will be a JavaScript wrapper for the native functions.  A configuration file describes what files and dependencies need to be added to the native project before building.  And of course the native source code that interacts with your JavaScript wrapper.

A normal DevKit plugin has a simple directory structure:

~~~
├── android
│   ├── GeolocPlugin.java
│   ├── config.json
│   ├── manifest.xml
│   └── manifest.xsl
├── ios
│   ├── GeoLoc.h
│   ├── GeoLoc.mm
│   └── config.json
├── js
│   ├── geoloc.js
│   └── native
│       └── geoloc.js
├── index.js
└── package.json
~~~

The `index.js` file is used by the addon system to notify the addon of `basil` events.  And the `package.json` file is a nodejs configuration file that specifies npm dependencies for the `index.js` source code.

The `android` and `ios` directories contain configuration and code for the supported native targets.

The `js` directory contains JavaScript files to include for each of the build targets.

#### ./index.js

The index JavaScript file can export several methods that get invoked by `basil` for different reasons:

+ `exports.init(common)` : Called when the addon is upgraded or first installed.

+ `exports.load(common)` : Called on all addons when the addon is loaded on `basil` startup before a build begins.

~~~
exports.init = function (common) {
	var logger = new common.Formatter('geoloc');
	logger.log("Initializing");
	exports.load();
};

exports.load = function (common) {
	var logger = new common.Formatter('geoloc');
	logger.log("Loading");
}
~~~

In practice, native plugins do not need to do anything in the index.js file.

#### ./js/

The JavaScript code can be specialized for each target based on where it is placed under this directory.  iOS, Android, and browser builds can all use different versions of your JS code.

During **iOS** builds, the order of JavaScript imports is:

+ `./js/ios/`
+ `./js/native/`
+ `./js/`

During **Android** builds, the order of JavaScript imports is:

+ `./js/android/`
+ `./js/native/`
+ `./js/`

During **browser** builds, the order of JavaScript imports is:

+ `./js/browser/`
+ `./js/`

Going with the geoloc example: When the Geolocation plugin is used in a game, it is imported with the line `import geoloc;`.  Where the `geoloc.js` file is read from follows the rules above.

For instance for an iOS build, the `./js/native/` directory will be added into the js.io compile path, meaning that JavaScript source files under `./js/native/` can be imported.

For the Geolocation plugin, the `./js/native/geoloc.js` file is imported by `import geoloc;` for iOS builds, which contains the JavaScript wrapper for the native code.  For browser builds, since the `./js/browser` directory is missing, the `./js/geoloc.js` file would be used, which does not contain the native wrapper code.

#### ./android/

Please see the [Android Plugin documentation](../native/android-plugin.html) for a description of the files under the `android` directory.

#### ./ios/

Please see the [iOS Plugin documentation](../native/ios-plugin.html) for a description of the files under the `ios` directory.

### Sending JavaScript Events to Native Code

To communicate with your native code from JavaScript, the `NATIVE.plugins.sendEvent` function is used to send data to the plugin, and the `NATIVE.events.registerHandler` function is used to receive data from the plugin.

For example, to send the GeoLocation plugin a request called "onRequest" with a JSON object as a parameter:

~~~
var e = {id:1, method:"getPosition", arguments:{}};
NATIVE.plugins.sendEvent("GeolocPlugin", "onRequest", JSON.stringify(e));
~~~

This causes the `onRequest` function in the `GeolocPlugin` native class to be invoked.

And to receive data from the plugin via the Events subsystem:

~~~
NATIVE.events.registerHandler('geoloc', function(evt) {
	if (evt.failed) {
	} else {
	}
});
~~~

##### Matching requests to responses

If it is necessary to match a request with the response from the plugin, it is good practice to include an `id` field in the request, which can be provided in the response to match it with the request.

This is not demonstrated here since GeoLocation does not need to match responses with requests.  A JavaScript array keyed with these ids is a good way to remember the callback functions to call when a response arrives.

### Android Integration

Please see the [Android Plugin documentation](../native/android-plugin.html).

### iOS Integration

Please see the [iOS Plugin documentation](../native/ios-plugin.html).
