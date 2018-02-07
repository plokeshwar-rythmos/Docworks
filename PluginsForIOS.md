Building Plugins for iOS
========================


This page describes [Native Code Plugins](Plugins) for the iOS platform.


Building an Application with a Native Plugin for iOS
----------------------------------------------------


1. Define your extern method in the C# file as follows:


````
[DllImport ("__Internal")]
private static extern float FooPluginFunction();
````

1. Set the editor to the iOS build target
1. Add your native code source files to the generated Xcode project's "Classes" folder (this folder is not overwritten when the project is updated, but don't forget to backup your native code).

If you are using C++ (.cpp) or Objective-C++ (.mm) to implement the plugin you must ensure the functions are declared with C linkage to avoid [name mangling issues](http://en.wikipedia.org/wiki/Name_mangling). 



````
extern "C" {
  float FooPluginFunction();
}
````

Plugins written in C or Objective-C do not need this since these languages do not use name-mangling.

Using Your Plugin from C#
-------------------------


iOS native plugins can be called only when deployed on the actual device, so it is recommended to wrap all native code methods with an additional C# code layer. This code should check Application.platform and call native methods only when the app is running on the device; dummy values can be returned when the app runs in the Editor. See the Bonjour browser sample application for an example.

Calling C# / JavaScript back from native code
---------------------------------------------

Unity iOS supports limited native-to-managed callback functionality via _UnitySendMessage_:

````
UnitySendMessage("GameObjectName1", "MethodName1", "Message to send");
````
This function has three parameters : the name of the target GameObject, the script method to call on that object and the message string to pass to the called method.

Known limitations:

1. Only script methods that correspond to the following signature can be called from native code: ` function MethodName(message:string)`
1. Calls to _UnitySendMessage_ are asynchronous and have a delay of one frame.

Automated plugin integration
----------------------------

Unity iOS supports automated plugin integration in a limited way. All files with extensions __.a__,__.m__,__.mm__,__.c__,__.cpp__ located in the Assets/__Plugins/iOS__ folder will be merged into the generated Xcode project automatically. However, merging is done by symlinking files from Assets/__Plugins/iOS__ to the final destination, which might affect some workflows. The __.h__ files are not included in the Xcode project tree, but they appear on the destination file system, thus allowing compilation of .m/.mm/.c/.cpp files.
 
**Note:** subfolders are currently not supported.

iOS Tips
--------


1. Managed-to-unmanaged calls are quite processor intensive on iOS. Try to avoid calling multiple native methods per frame.
1. As mentioned above, wrap your native methods with an additional C# layer that calls native code on the device and returns dummy values in the Editor.
1. String values returned from a native method should be UTF-8 encoded and allocated on the heap. Mono marshaling calls are free for strings like this.
1. As mentioned above, the Xcode project's "Classes" folder is a good place to store your native code because it is not overwritten when the project is updated.
1. Another good place for storing native code is the Assets folder or one of its subfolders. Just add references from the Xcode project to the native code files: right click on the "Classes" subfolder and choose "Add-&gt;Existing files...".


Examples
--------



###Bonjour Browser Sample
A simple example of the use of a native code plugin can be found [here](../uploads/Examples/iPhoneNativeCodeSample.zip)

This sample demonstrates how objective-C code can be invoked
from a Unity iOS application. This application implements a very simple Bonjour client.
The application consists of a Unity iOS project (Plugins/Bonjour.cs is the C# interface to the native code, while BonjourTest.js is the JS script that implements the application logic) and native code (Assets/Code) 
that should be added to the built Xcode project.
