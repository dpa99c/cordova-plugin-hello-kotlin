cordova-plugin-hello-c
======================

A simple example of a Cordova plugin that uses pure C code.

It illustrates how to use platform-specific (either Android or iOS) C code and how to share C code cross-platform (between Android and iOS).

For Android it utilizes the Android NDK to compile architecture-specific libraries and a JNI wrapper to expose the C functions to the Java plugin API.

For iOS it uses the pure C source code in place alongside the Objective-C plugin wrapper, as well as an example cross-platform library compiled as a static library for iOS.

# usage example

- Clone the [test project](https://github.com/dpa99c/cordova-plugin-hello-c-test)
- Add Android and iOS platforms: `cordova platform add android && cordova platform add ios`
- Run: `cordova run android` / `cordova run ios`

# Plugin structure

- `plugin.xml` - Specifies the plugin's source files and libraries to copy to the platforms of the Cordova project into which the plugin is installed
- `compile-android` and `compile-android.cmd` - Script to recompile C source code as a shared library for use with Android platform
- `compile-ios` - Script to recompile cross-platform C library as static library for use with iOS platform
- `src/` - C source code and build scripts
    - `common/` - cross-platform C source code to run on both Android and iOS
        - `mylib/` - example cross-platform C library
    - `android/` - source code and build scripts for Android platform
        - `build-extras.gradle` - Gradle build script to link up the NDK make file for debugging C code in Android Studio
        - `HelloCPlugin.java` - Provides the native Java implementation for Android of the Cordova plugin
        - `HelloCJni.java` - Provides the Java interface to the underlying JNI C implementation
            - `jni/` - JNI C implementation and build script
                - `HelloJni.c` - C implementation of the JNI interface defined by `HelloCJni.java`, including 
                    - Android-specific implementation to get current CPU architecture
                    - interfaces to cross-platform C functionality
                - `Android.mk` - NDK Make script to build the C source code into architecture-specific shared libraries
            - `libs/` - contains folders for the various architecture-specific shared libraries built by the NDK Make script
    - `ios/` - source code and build scripts for iOS platform
        - `c_getArch.c` & `c_getArch.h` - iOS-specific implementation to get current CPU architecture
        - `HelloCPlugin.c` & `HelloCPlugin.h` - Provides the native Objective-C implementation for iOS of the Cordova plugin
        - `ios_compile.sh` - Script to compile the cross-platform example library in `src/common/mylib/` as a static library
        - `Makefile` - Make script invoked by the above script to perform the C compilation.
        - `libs/` - the compiled cross-platform library
            - `libmylib.a` - the compiled cross-platform library as a multi-architecture static library
            - `headers/` - the header files of the cross-platform library (static libraries require the headers externally) 
        

# Recompiling libraries
If you modify the C source files, be sure to re-build the compiled libraries.

## Android

You can re-build the `libhelloc.so` binaries using the ndk-build script.

To do so:

- Install Android NDK as [instructed here](https://developer.android.com/ndk/guides/index.html)
- Add the NDK install path to your path environment variable
    - By default it's installed under $ANDROID_SDK_HOME/ndk-bundle
    - e.g. `export PATH=$PATH;$ANDROID_SDK_HOME/ndk-bundle`
- Set the ANDROID_NDK_HOME environment variable to your NDK install path
    - e.g. `export ANDROID_NDK_HOME=$ANDROID_SDK_HOME/ndk-bundle`
- Open terminal in plugin root folder
- Run `./compile-android` (`compile-android.cmd` on Windows)

If you are editing the C source code of the plugin in place in the example project:

- Modify the C source in `plugins/cordova-plugin-hello-c/src/android/jni` or `plugins/cordova-plugin-hello-c/src/common`
- Open terminal in `plugins/cordova-plugin-hello-c`
- Run `compile-android` (`compile-android.cmd` on Windows)
- From the project root, remove and re-add the android platform to apply the plugin changes to the project
    `cordova platform rm android && cordova platform add android`
    
## iOS
If you modify the C source code in `common/mylib/` you'll need to rebuild the static library and headers in `src/ios/libs`.

- Open terminal in plugin root folder
- Run `./compile-ios`

If you are editing the C source code of the plugin in place in the example project:

- Modify the C source in `plugins/cordova-plugin-hello-c/src/ios/` or `plugins/cordova-plugin-hello-c/src/common`
- Open terminal in `plugins/cordova-plugin-hello-c`
- Run `./compile-ios`
- From the project root, remove and re-add the platform to apply the plugin changes to the project
    `cordova platform rm ios && cordova platform add ios`

# Debugging C source code
 
## Android

- The Android NDK enables C/C++ source code to be debugged in Android Studio alongside Java.
- To do so, the source code must be included but **not** the compiled libraries.
- To debug this plugin in Android Studio do the following:
    - Edit `plugin.xml` and in the `<platform name="android">` block, comment out the source-file lines in the PRODUCTION block which include the compiled libraries
    - Remove/re-add the plugin or Android platform in your project to update the plugin files in the platform project
    - Open the Android platform project (`platforms/android`) in Android Studio
    - Connect an Android device for debugging
    - Use the Project Explorer to find and open one of the `.c` source files
    - Place a breakpoint, for example on a `return` statement
    - Select "Run" > "Debug ..." from the menu

## iOS

- Since iOS is a C-based platform, C debugging is inherently supported in the Xcode IDE
- However, to debug the C code in the static library `src/ios/libs/libmylib.a`, you'll need to comment out the library files and comment in the source code for it:
- Edit `plugin.xml` and in the `<platform name="ios">` block
    - comment out the lines in the PRODUCTION block which include the compiled library and headers.
    - comment in the the commented-out lines in the DEBUG block which will include the uncompiled C source code for the library
- Remove/re-add the plugin or iOS platform in your project to update the plugin files in the platform project
- Use the OSX Finder to browse to `platforms/ios/` in your Cordova project and double-click the `.xcodeproj` file to open it in Xcode.