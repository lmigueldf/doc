# Building for Android

## Overview

Testing your game on your cellphone for the first time is an exhilerating
affair full of nail-biting anticipation. The process of getting there for
Android devices has been streamlined for you in the Game Closure DevKit.

Building your game to run on a mobile device is done using the Game Closure
DevKit `devkit` command-line tool.  See the [Android Setup
Guide](./android-setup.html) for instructions on how to get prerequisites,
supported mobile devices, and steps on getting started.

## Building an .APK

To build your game for debugging, open a console and change the
directory to your game's top-level folder. For example, after cloning
the whackthatmole getting started project:

~~~
$ cd whackthatmole
$ ls
README.md     manifest.json resources     src
$ devkit debug native-android --no-compress --debug --clean
[build]   Building: debug/native-android
...
~~~

By specifying the --no-compress option, devkit will not try to compress
JavaScript files.  This speeds up the build process and is recommended for most
pre-release builds.

Combining the --no-compress and --debug options will also allow you to read the
logs from the device and do
[remote JavaScript debugging](./android-remote-debug.html) from your computer.


If you want to build your game for the Google Play store use the command

~~~
$ devkit release native-android
~~~

When the build completes, you will see a line such as:

~~~
[android]   saved to /Users/gc/sandbox/whackthatmole/build/release/native-android/whackthatmole.apk
~~~

This indicates where the output APK file has been saved.  You can then install
it to a connected device.

## Installing to a Connected Device

Install an application to a device connected via a USB data cable using the
Android Debug Bridge (ADB):

`$ adb install -r <path-to-apk>`

The -r flag means to attempt a reinstall.  Note that this will fail if the APK
is signed differently (mixing debug and release versions) so you may need to
uninstall your game from the phone to reinstall.

## Troubleshooting Issues

If you run into problems, try our
[Android troubleshooting guide](./android-troubleshooting.html).

### Appendix: Build Options

General build options:

+ `--help` / `-h` : Display this help menu
+ `--clean` / `-c` : Clean build before compilation
+ `--compress` : Compress JavaScript during build (Default)
+ `--no-compress` : Do not compress JavaScript during build
+ `--repack` / `--js-only` : Only build the JavaScript
+ `--resources-only` : Skip the native build
+ `--clearstorage` / `s` : Clear localStorage on device
+ `--install` : Install after build
+ `--open` : Install and open after build
+ `--enableReleaseLogging` : Print logs in release mode (debug mode always prints logs)

### Appendix: Example Usage

**For a fast debug build:**

~~~
$ devkit debug native-android
$ adb install -r build/debug/native-android/whackthatmole.apk
~~~

This specifies a debug build, cleans before building, does not compress JavaScript, and can be used with the remote JavaScript debugger.

**For a thorough release build:**

~~~
$ devkit release native-android
$ adb install -r build/release/native-android/whackthatmole.apk
~~~

This specifies a release build, cleans before building, compresses JavaScript, and will sign the resulting APK for release.
