iOS Specific Optimizations
==========================


This page details optimizations which are unique to iOS deployment. 

Script Call Optimization
------------------------

Most of the functions in the __UnityEngine__ namespace are implemented in C/C++. Calling a C/C++ function from a Mono script involves a performance overhead, so you can save to save about 1 to 4 milliseconds per frame using iOS Script Call optimization. 

To access iOS Script Call optimization, navigate to the [Player Settings](class-PlayerSettings) window (menu: __Edit__ >  __Project Settings__ > __Player__) and select the iOS icon (shown below).


![iOS Icon - select this to access the iOS settings in the Player Settings window](../uploads/Main/iosoptimizations_icon2.png)


Locate the __Script Call Optimization*__ setting in the __Other Settings_ section.

The options for this setting are:-

* __Slow and Safe__ - the default Mono internal call handling with exception support.
* __Fast but no exceptions__ - a faster implementation of Mono internal call handling. However, this doesn't support exceptions and so should be used with caution. An app that doesn't explicitly handle exceptions (and doesn't need to deal with them gracefully) is an ideal candidate for this option.

Setting the Desired Framerate
-----------------------------

Unity iOS allows you to change the frequency with which your application will try to execute its rendering loop, which is set to 30 frames per second by default. You can lower this number to save battery power but of course this saving will come at the expense of frame updates. Conversely, you can increase the framerate to give the rendering priority over other activities such as touch input and accelerometer processing. You will need to experiment with your choice of framerate to determine how it affects gameplay in your case.

If your application involves heavy computation or rendering and can maintain only 15 frames per second, say, then setting the desired frame rate higher than fifteen wouldn't give any extra performance. The application has to be optimized sufficiently to allow for a higher framerate.

To set the desired framerate, change [Application.targetFrameRate](ScriptRef:Application-targetFrameRate.html).

Tuning Accelerometer Processing Frequency
-----------------------------------------

If accelerometer input is processed too frequently then the overall performance of your game may suffer as a result. By default, a Unity iOS application will sample the accelerometer 60 times per second. You may see some performance benefit by reducing the accelerometer sampling frequency and it can even be set to zero for games that don't use accelerometer input. You can change the accelerometer frequency from the __Other Settings__ panel in the [iOS Player Settings](class-PlayerSettings).
