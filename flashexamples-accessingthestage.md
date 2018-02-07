Example: Accessing the Stage
============================


You can access the [Flash Stage](http://help.adobe.com/en_US/FlashPlatform/reference/actionscript/3/flash/display/Stage.html) from your C#/JS scripts in the following way:



````
ActionScript.Import("com.unity.UnityNative"); 
ActionScript.Statement("trace(UnityNative.stage);");


````


As an example, the following C# code will output the flashvars supplied to a SWF:


````
ActionScript.Import("com.unity.UnityNative"); 
ActionScript.Statement(
    "var params:Object = LoaderInfo(UnityNative.stage.loaderInfo).parameters;" +
    "var key:String;" +
    "for (key in params) {" +
        "trace(key + '=' + params[key]);" +
    "}"
);


````
