# Stages of Android Development

## Overview

Android cellphones are a friendly and flexible platform for game development,
with a wide variety of mobile devices supporting it.  Using the Game
Closure DevKit you may target your HTML5 games to run smoothly across
all of the popular Android devices without worrying about specific
hardware or OS differences between the devices.

Based on what stage of development your game is in, you may consider different
types of builds:

### 1. Web Development

Test your code, art, and sound assets entirely in a web browser simulating a
mobile device.  Quickly iterate using the Game Closure DevKit web interface
without doing any setup required for building on the mobile device.

Investigate coding errors, step through running code, pause execution, and
inspect code objects using the Chrome built-in Web Inspector as with normal
HTML5 game development.  Improve performance by finding code hot-spots with the
CPU Profiler to target optimization.  Hunt down memory leaks using the Heap
Profiler to identify object pile-ups.

<div class="figure-wrapper">
<figure>
<img src="./assets/game-walkthrough/game-console.png"></img>
<figcaption>DevKit Device Emulator.</figcaption>
</figure>
</div>

For more information see the [basic DevKit guides](../guide/install.html).

### 2. Test on Device

Test your game code live on the device with the [Native
Inspector](./native-remote-debug). Follow the instructions to install
and run the NativeInspector, then [create a debug build](./android-build.html)
with `devkit debug native-android`. You can also include the `--open` flag
to automatically install the app on the connected device and start it when
ready.

When connected to the Native Inspector, the same Chrome Web Inspector interface
may be used to investigate coding errors on the mobile device from the comfort
of your computer browser.

You may also use `adb logcat | grep JS` to view the JavaScript logs.

At this stage the mobile device no longer needs to be tethered to your computer
and you can show off your game!

### 3. Release Builds

[Build and install](./android-build.html) your game to a device connected via
USB.  Simply run `devkit release native-android` to produce a release APK,
then install it with `adb install`.

### 4. Deployment

The same release APK that you built for testing may be uploaded to the Google
Play Store.

