Cordova Plugin Example
======================

This is sample cordova plugin (currently for android platform only). Tested with Cordova 3.4 + Android 4.2.2

How to run example?
-------------------

To add this plugin to your cordova project, type:

	cordova plugin add https://github.com/usefulio/cordova-example-plugin.git

And include SamplePlugin.js in your www/index.html  

	<script type="text/javascript" src="plugins/SamplePlugin.js"></script>

This will expose example function `sum` to your cordova box. Function receives two integers and returns their sum.
**Pretty useless, but it demonstrates how to connect your javascript code with native layer - sum is calculated by native code outside of cordova box.**

To call `sum` from javascript code write:

	window.sum(2, 3, function(result) { alert("Result:" + result); }, function(err) { alert(err); });

First two parameters are numbers to sum, then two functions: success callback and error callback. In this example, page shuld alert "Result: 5". 

Here is full `www/index.html`:

	<!DOCTYPE html>
	<html>
		<head>
			<meta charset="utf-8" />
			<title>Example</title>

			<script type="text/javascript" src="cordova.js"></script>
			<script type="text/javascript" src="plugins/SamplePlugin.js"></script>

			<script type="text/javascript">
				function onLoad() {
					document.addEventListener('deviceready', onDeviceReady, false);
				}

				function onDeviceReady() {
					window.sum(2, 3, function(result) { alert("Result:" + result); }, function(err) { alert(err); });
				}
			</script>
		</head>

		<body onload="onLoad()">
			<h1>Sample plugin</h1>
		</body>
	</html>

How it works?
-------------

We have **three files**: native code (.java), javascript code which exposes native code to cordova box (.js) and plugin manifest file (.xml). 
<br />

Let's take a look at plugin files:

`src/android/com/usefulio/plugin/SamplePlugin.java` contains native code which executes outside of your cordova box.
<br />
`www/SamplePlugin.js` connects native code with javascript - declares javascript function `sum`. 
<br />
`plugin.xml` contains info about plugin itself and paths to java and js files.


SamplePlugin.java
-----------------

	package com.usefulio.plugin;

	import org.apache.cordova.CordovaPlugin;
	import org.apache.cordova.CallbackContext;
	import org.apache.cordova.PluginResult;

	import org.json.JSONArray;
	import org.json.JSONException;
	import org.json.JSONObject;

	/**
	 * This class performs sum called from JavaScript.
	 */
	public class SamplePlugin extends CordovaPlugin {
		@Override
		public boolean execute(String action, JSONArray args, CallbackContext callbackContext) throws JSONException {
			if (action.equals("sum")) {
				Integer num1 = args.getInt(0);
				Integer num2 = args.getInt(1);
				this.sum(num1, num2, callbackContext);
				return true;
			}
			return false;
		}

		private void sum(Integer num1, Integer num2, CallbackContext callbackContext) {
			if(num1 != null && num2 != null) {
				callbackContext.success(num1 + num2);
			} else {
				callbackContext.error("Expected two integer arguments.");
			}
		}
	}

First method `execute` is the same in every plugin, it overrides standard method of CordovaPlugin class and allways has the same parameters:

- Parameter `action` is string containing name of method to execute.
- `args` is array containing variable number of arguments
- `callbackContext` has `.success` and `.error` methods which are called when your function finishes or in case of error.

Note that our plugin class can have many methods, but just one `execute` function. We test `action` argument to see which method we should call. 
You can extract method arguments from JSONArray with `.getInt(argumentIndex)`, `.getString(argumentIndex)` etc.
<br />

Next method `sum` is our example function: it receives two integers and callbackContext. In case of error it calls `callbackContext.error` with error message string, or in case of success it calls `callbackContext.success` with result.

SamplePlugin.js
---------------

	window.sum = function(num1, num2, successCallback, errorCallback) {
		cordova.exec(successCallback, errorCallback, "SamplePlugin", "sum", [num1, num2]);
	};

In this file we extend `window` object with function `sum` which calls `cordova.exec`. Cordova translates that and calls `execute` method from our class declared in SamplePlugin.java. 

Arguments to `exec` are allways the same: two callback functions (success and error callback), plugin class name, action name and array of arguments.
You can pass any number of arguments - they are processed by execute function as described.

plugin.xml
----------

	<?xml version="1.0" encoding="UTF-8"?>
	<plugin xmlns="http://www.phonegap.com/ns/plugins/1.0"
		id="com.usefulio.plugin.SamplePlugin"
		version="1.0.0">

		<name>SamplePlugin</name>

		<description>
		Cordova plugin example
		</description>

		<asset src="www/SamplePlugin.js" target="plugins/SamplePlugin.js" />

		<engines>
			<engine name="cordova" version=">=3.0.0" />
		</engines>

		<!-- android -->
		<platform name="android">
			<config-file target="res/xml/config.xml" parent="/*">
				<feature name="SamplePlugin">
					<param name="android-package" value="com.usefulio.plugin.SamplePlugin"/>
				</feature>
			</config-file>

			<source-file src="src/android/com/usefulio/plugin/SamplePlugin.java" target-dir="src/com/usefulio/plugin" />
		</platform>

		<!-- more platforms here -->

	</plugin>

This file tells cordova how to deal with plugin. Package name, class name and (relative) paths to files are defined here. More details about plugin.xml [here](http://docs.phonegap.com/en/3.4.0/plugin_ref_spec.md.html#Plugin%20Specification).

For more details refer:

[Cordova Plugin Development Guide](http://docs.phonegap.com/en/3.4.0/guide_hybrid_plugins_index.md.html#Plugin%20Development%20Guide)
<br />
and
<br />
[Android Plugins](http://docs.phonegap.com/en/3.4.0/guide_platforms_android_plugin.md.html#Android%20Plugins)
<br />
<br />
That's it :)
