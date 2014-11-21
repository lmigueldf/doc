# DevKit Native Plugins

The Game Closure DevKit supports a plugin system that enables you to develop
unique features for your game in native code and JavaScript that will
seamlessly interoperate with and persist through upgrades to new versions.

+ These plugins can be easily shared privately with your internal development
team or shared back to Game Closure to benefit the rest of the HTML5 game
development community.

+ Plugins allow your game to use a minimum set of permissions, which will
prevent users from getting scared off.

+ Native plugin code has access to the `manifest.json` file contents so you can
configure the plugins through that file.

+ Native code and JavaScript interact through a message passing interface.

The JS Event System is used to send events to JavaScript, and a function is
used to deliver events from JavaScript to native code.  This native interface
is the same for both iOS and Android.

## Plugin Example: GeoLocation

To demonstrate the plugin system, take the [GeoLocation
Plugin](https://github.com/gameclosure/geoloc) as an example.  It will be
referenced throughout the documentation.

To install the Geolocation module, run
`devkit install https://github.com/gameclosure/geoloc`.  It will be
installed in your game folder under `modules/geoloc`.

This will also call `npm install` inside the module folder, which means
you should use the npm scripts feature when you need custom behavior
when installing (example: running `android update`). See the [npm
docs](https://www.npmjs.org/doc/misc/npm-scripts.html) for details.

## Using plugins from your game

Furthermore at the top of the game's `src/Application.js`:

~~~
import geoloc;
~~~

This imports the file specified in the package.json file under the 'js' key.
This is generally the name of the plugin, but can vary depending on the settings
(or to prevent conflicts) -- check the plugin's README.md for details.


## Module Directory Structure

The plugin system for iOS consists of several pieces:  There will be a
JavaScript wrapper for the native functions.  A configuration file describes
what files and dependencies need to be added to the native project before
building.  And of course there is the native source code that interacts with
your JavaScript wrapper.

A normal DevKit module has a simple directory structure:

~~~
├── README.md
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
│   └── index.js
└── package.json
~~~

All plugins should have a `README.md` or similar documentation to describe how
to install it, the correct path for importing it,  and platform-specific
considerations when using the plugin.

#### ./package.json
The `package.json` file is a nodejs configuration file that
specifies npm dependencies for the javascript source code and defines the
paths you can use to import the plugin, as well as where the platform-specific
code can be located.

Example package.json from the flurry module:
```
{
    "name": "devkit-flurry",
    "version": "0.0.1",
    "devkit": {
        "clientPaths": {
            "flurry": "js"
        },
        "extensions": {
            "android": "android",
            "ios": "ios"
        }
    }
}
```

The `clientPaths` map defines the import paths used to import the plugin
from your javascript, as well as where that javascript is located. In
this example, `import flurry` will try to import index.js inside the js
folder.

Notice that this also specifies where to find the android and ios native
code using the extensions key.

#### ./js/

The `js` directory contains JavaScript files that will be importable in the
app.  In the case of geoloc, this is `import geoloc;`
to import the `js/index.js` file.

You do not need to add a plugin to a game's `manifest.json` "ection to
use JavaScript imports that it provides.

#### ./android/

Please see the [Android Plugin documentation](../native/android-plugin.html)
for a description of the files under the `android` directory.

#### ./ios/

Please see the [iOS Plugin documentation](../native/ios-plugin.html) for a
description of the files under the `ios` directory.

### Sending JavaScript Events to Native Code

To communicate with your native code from JavaScript, the
`NATIVE.plugins.sendEvent` and `NATIVE.plugins.sendRequest` functions can
be used to send data to the plugin.

The `NATIVE.events.registerHandler` function can be used to register a handler
function to receive global events published by the plugin.


`NATIVE.plugins.sendEvent` accepts three arguments - the plugin name, the event
name (corresponding to the native function that handles it), and a JSON
stringified data object with any additional info to send to the plugin.

For example, to send the GeoLocation plugin a request called "onRequest" with a
JSON object as a parameter:

~~~
var e = {id:1, method:"getPosition", arguments:{}};
NATIVE.plugins.sendEvent("GeolocPlugin", "onRequest", JSON.stringify(e));
~~~

This causes the `onRequest` function in the `GeolocPlugin` native class to be
invoked.


`NATIVE.plugins.sendRequest` can be used in a similar way to send an event to
the plugin while expecting a response. sendRequest passes a callback to the
plugin which gets called in the standard (err, response) format.

`NATIVE.plugins.sendRequest` accepts four arguments - the plugin name,
the event name (corresponding to the native function that handles it), an object
of data to be attached to the event (not stringified!), and the callback.

~~~
var data = {id:1, method:"getPosition", arguments:{}};
NATIVE.plugins.sendRequest("GeolocPlugin", "onRequest", data, function (err, res) {
    if (!err) {
        console.log("response from geoloc plugin!", res);
    }
});
~~~



Another way to improve the readability of your code is to split the
`NATIVE.plugins.sendEvent` call into a convenient wrapper.  In the following
example from the [Facebook plugin](http://github.com/gameclosure/facebook/),
`pluginSend("login");` can be used instead of the long-hand form above:

~~~
function pluginSend(evt, params) {
    NATIVE && NATIVE.plugins.sendEvent("FacebookPlugin", evt,
                JSON.stringify(params || {}));
}

var Facebook = Class(function () {
    this.login = function() {
        pluginSend("login");
    };
});

exports = new Facebook();
~~~

### Receiving JavaScript Events from Native Code

To receive data from the plugin via the Events subsystem:

~~~
NATIVE.events.registerHandler('geoloc', function(evt) {
    if (evt.failed) {
    } else {
    }
});
~~~


You can also invoke native methods 

The following `pluginOn` and `invokeCallbacks` functions are helpful wrappers
from the [Facebook plugin](http://github.com/gameclosure/facebook/) you can
drop into your own plugin to easily handle callbacks from native code:

~~~
function pluginOn(evt, next) {
    NATIVE && NATIVE.events.registerHandler(evt, next);
}

function invokeCallbacks(list, clear) {
    // Pop off the first two arguments and keep the rest
    var args = Array.prototype.slice.call(arguments);
    args.shift();
    args.shift();

    // For each callback,
    for (var ii = 0; ii < list.length; ++ii) {
        var next = list[ii];

        // If callback was actually specified,
        if (next) {
            // Run it
            next.apply(null, args);
        }
    }

    // If asked to clear the list too,
    if (clear) {
        list.length = 0;
    }
}

var Facebook = Class(function () {
    var loginCB = [];

    this.init = function(opts) {
        pluginOn("facebookState", function(evt) {
            invokeCallbacks(loginCB, true, evt.state === "open");
        });
    }

    this.login = function(next) {
        loginCB.push(next);

        pluginSend("login");
    };
});

exports = new Facebook();
~~~

##### Matching requests to responses

If it is necessary to match a request with the response from the plugin, it is
good practice to include an `id` field in the request, which can be provided in
the response to match it with the request.

This is not demonstrated here since GeoLocation does not need to match
responses with requests.  A JavaScript array keyed with these ids is a good way
to remember the callback functions to call when a response arrives.

### Plugin JavaScript Design Recommendations

Note that the `geoloc` plugin is a special case that breaks out of the normal
js.io class system to add features to the global `navigator` object.  The
preferred way to develop module JavaScript is by using the normal js.io class
system.

For example, for the [MoPub](https://github.com/gameclosure/mopub/) plugin
there is a `js/index.js` file:

~~~
var MoPub = Class(function () {
    this.showInterstitial = function () {
        logger.log("{moPub} Showing interstitial");

        NATIVE && NATIVE.plugins.sendEvent("MoPubPlugin", "showInterstitial",
                JSON.stringify({}));
        }
    };
});

exports = new MoPub();
~~~

The JS object is imported in games that use it with:

~~~
import mopub as moPub;
~~~

And then games can use the `showInterstitial` method:

~~~
moPub.showInterstitial();
~~~

### Android Integration

Please see the [Android Plugin documentation](../native/android-plugin.html).

### iOS Integration

Please see the [iOS Plugin documentation](../native/ios-plugin.html).
