Text Asset
==========


__Text Assets__ are a format for imported text files. When you drop a text file into your Project Folder, it will be converted to a Text Asset. The supported text formats are:


* **.txt**
* **.html**
* **.htm**
* **.xml**
* **.bytes**
* **.json**
* **.csv**
* **.yaml**
* **.fnt**

*Note that script files are also considered text assets for the purposes of using the [AssetDatabase.FindAssets](ScriptRef:AssetDatabase.FindAssets) function, so they will also be included in the list of results when this function is used with the "t:TextAsset" filter.*

![The Text Asset Inspector](../uploads/Main/TextAssetInspector.png) 


Properties
----------



|**_Property:_** |**_Function:_** |
|:---|:---|
|__Text__ |The full text of the asset as a single string. |


Details
-------


The Text Asset is a very specialized use case. It is extremely useful for getting text from different text files into your game while you are building it. You can write up a simple .txt file and bring the text into your game very easily. It is not intended for text file generation at runtime. For that you will need to use traditional Input/Output programming techniques to read and write external files.

Consider the following scenario. You are making a traditional text-heavy adventure game. For production simplicity, you want to break up all the text in the game into the different rooms. In this case you would make one text file that contains all the text that will be used in one room. From there it is easy to make a reference to the correct Text Asset for the room you enter. Then with some customized parsing logic, you can manage a large amount of text very easily.


###Binary data
A special feature of the text asset is that it can be used to store binary data. By giving a file the extension **.bytes** it can be loaded as a text asset and the data can be accessed through the __bytes__ property.

For example put a jpeg file into the Resources folder and change the extension to **.bytes**, then use the following script code to read the data at runtime:


````
//Load texture from disk
TextAsset bindata= Resources.Load("Texture") as TextAsset;
Texture2D tex = new Texture2D(1,1);
tex.LoadImage(bindata.bytes); 
````

Please notice that files with the **.txt** and **.bytes** extension will be treated as text and binary files, respectively. Do not attempt to store a binary file using the **.txt** extension, as this will create unexpected behaviour when attempting to read data from it.

Hints
-----


* Text Assets are serialized like all other assets in a build. There is no physical text file included when you publish your game.
* Text Assets are not intended to be used for text file generation at runtime.
