cordova-plugin-hello-kotlin
===========================

A simple example of a Cordova plugin that uses Kotlin (instead of Java) to implement the native Cordova plugin interface on Android.

Since `cordova-android@7` currently doesn't implicitly support Kotlin, this plugin uses hook scripts to add the necessary native config to make the Kotlin work in the generated Android project and to remove the Kotlin source files from the native Android project on uninstalling the plugin.


- Clone the [test project](https://github.com/dpa99c/cordova-plugin-hello-kotlin-test): `git clone https://github.com/dpa99c/cordova-plugin-hello-kotlin-test`
- Add the Android platform: `cordova platform add android`
- Run the test app: `cordova run android`