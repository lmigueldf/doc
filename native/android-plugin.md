# Android Plugin System

Writing the Android native code for a DevKit plugin is covered in this guide, which uses the [Geolocation plugin](http://github.com/gameclosure/geoloc) as an example.

For an overview of the plugin system, the requisite directory structure, and how to write the JavaScript component of the plugin, please see the [main plugin development guide](../guide/plugins.html).

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

