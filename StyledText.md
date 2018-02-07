#Rich Text

The text for UI elements and text meshes can incorporate multiple font styles and sizes. Rich text is supported both for the UI System and the legacy GUI system. The Text, GUIStyle, GUIText and TextMesh classes have a __Rich Text__ setting which instructs Unity to look for markup tags within the text. The [Debug.Log](ScriptRef:Debug.Log.html) function can also use these markup tags to enhance error reports from code. The tags are not displayed but indicate style changes to be applied to the text.

##Markup format

The markup system is inspired by HTML but isn't intended to be strictly compatible with standard HTML. The basic idea is that a section of text can be enclosed inside a pair of matching tags:-

&#160;&#160;&#160;We are &lt;b&gt;not&lt;/b&gt; amused

As the example shows, the tags are just pieces of text inside the "angle bracket" characters, &lt; and &gt;. The text inside the tag denotes its name (which in this case is just **b**). Note that the tag at the end of the section has the same name as the one at the start but with the slash / character added. The tags are not displayed to the user directly but are interpreted as instructions for styling the text they enclose. The b tag used in the example above applies boldface to the word "not", so the text will appear onscreen as:-

&#160;&#160;&#160;We are **not** amused

A marked up section of text (including the tags that enclose it) is referred to as an **element**.


###Nested elements

It is possible to apply more than one style to a section of text by "nesting" one element inside another

&#160;&#160;&#160;We are &lt;b&gt;&lt;i&gt;definitely not&lt;/i&gt;&lt;/b&gt; amused

The i tag applies italic style, so this would be presented onscreen as

&#160;&#160;&#160;We are **_definitely not_** amused

Note the ordering of the ending tags, which is in reverse to that of the starting tags. The reason for this is perhaps clearer when you consider that the inner tags need not span the whole text of the outermost element

&#160;&#160;&#160;We are &lt;b&gt;absolutely &lt;i&gt;definitely&lt;/i&gt; not&lt;/b&gt; amused

which gives

&#160;&#160;&#160;We are **absolutely _definitely_ not** amused


###Tag parameters

Some tags have a simple all-or-nothing effect on the text but others might allow for variations. For example, the **color** tag needs to know which color to apply. Information like this is added to tags by the use of **parameters**:-

&#160;&#160;&#160;We are &lt;color=green&gt;green&lt;/color&gt; with envy

Note that the ending tag doesn't include the parameter value. Optionally, the value can be surrounded by quotation marks but this isn't required.


##Supported tags

The following list describes all the styling tags supported by Unity.

|**_Tag_** |**_Description_** |**_Example_** |**_Notes_** |
|:---|:---|:---|
| **b**| Renders the text in boldface.| &#160;&#160;&#160;We are &lt;b&gt;not&lt;/b&gt; amused.| | 
| **i**| Renders the text in italics.| &#160;&#160;&#160;We are &lt;i&gt;usually&lt;/i&gt; not amused.| | 
| **size**| Sets the size of the text according to the parameter value, given in pixels.| &#160;&#160;&#160;We are &lt;size=50&gt;largely&lt;/size&gt; unaffected.| Although this tag is available for Debug.Log, you will find that the line spacing in the window bar and Console looks strange if the size is set too large.| 
| **color**| Sets the color of the text according to the parameter value. The color can be specified in the traditional HTML format. _&#160;&#160;&#160;\#rrggbbaa_ ...where the letters correspond to pairs of hexadecimal digits denoting the red, green, blue and alpha (transparency) values for the color. For example, cyan at full opacity would be specified by | _&#160;&#160;&#160;&lt;color=\#00ffffff&gt;..._ | Another option is to use the name of the color. This is easier to understand but naturally, the range of colors is limited and full opacity is always assumed. _&#160;&#160;&#160;&lt;color=cyan&gt;..._  The available color names are given in the table below.| 

|**_Color name_** |**_Hex value_** |**_Swatch_** |
|:---|:---|:--|
|aqua (same as cyan)|`#00ffffff`|![](../uploads/Main/CyanSwatch.png)|
|black|`#000000ff`|![](../uploads/Main/BlackSwatch.png)|
|blue|`#0000ffff`|![](../uploads/Main/BlueSwatch.png)|
|brown|`#a52a2aff`|![](../uploads/Main/BrownSwatch.png)|
|cyan (same as aqua)|`#00ffffff`|![](../uploads/Main/CyanSwatch.png)|
|darkblue|`#0000a0ff`|![](../uploads/Main/DarkblueSwatch.png)|
|fuchsia (same as magenta)|`#ff00ffff`|![](../uploads/Main/MagentaSwatch.png)|
|green|`#008000ff`|![](../uploads/Main/GreenSwatch.png)|
|grey|`#808080ff`|![](../uploads/Main/GreySwatch.png)|
|lightblue|`#add8e6ff`|![](../uploads/Main/LightblueSwatch.png)|
|lime|`#00ff00ff`|![](../uploads/Main/LimeSwatch.png)|
|magenta (same as fuchsia)|`#ff00ffff`|![](../uploads/Main/MagentaSwatch.png)|
|maroon|`#800000ff`|![](../uploads/Main/MaroonSwatch.png)|
|navy|`#000080ff`|![](../uploads/Main/NavySwatch.png)|
|olive|`#808000ff`|![](../uploads/Main/OliveSwatch.png)|
|orange|`#ffa500ff`|![](../uploads/Main/OrangeSwatch.png)|
|purple|`#800080ff`|![](../uploads/Main/PurpleSwatch.png)|
|red|`#ff0000ff`|![](../uploads/Main/RedSwatch.png)|
|silver|`#c0c0c0ff`|![](../uploads/Main/SilverSwatch.png)|
|teal|`#008080ff`|![](../uploads/Main/TealSwatch.png)|
|white|`#ffffffff`|![](../uploads/Main/WhiteSwatch.png)|
|yellow|`#ffff00ff`|![](../uploads/Main/YellowSwatch.png)|


**material**

This is only useful for text meshes and renders a section of text with a material specified by the parameter. The value is an index into the text mesh's array of materials as shown by the inspector.

&#160;&#160;&#160;We are &lt;material=2&gt;texturally&lt;/material&gt; amused



**quad**

This is only useful for text meshes and renders an image inline with the text. It takes parameters that specify the material to use for the image, the image height in pixels, and a further four that denote a rectangular area of the image to display. Unlike the other tags, quad does not surround a piece of text and so there is no ending tag - the slash character is placed at the end of the initial tag to indicate that it is "self-closing".

&#160;&#160;&#160;&lt;quad material=1 size=20 x=0.1 y=0.1 width=0.5 height=0.5 /&gt;

This selects the material at position in the renderer's material array and sets the height of the image to 20 pixels. The rectangular area of image starts at given by the x, y, width and height values, which are all given as a fraction of the unscaled width and height of the texture.


##Editor GUI

Rich text is disabled by default in the editor GUI system but it can be enabled explicitly using a custom GUIStyle. The richText property should be set to true and the style passed to the GUI function in question:-

````
GUIStyle style = new GUIStyle ();
style.richText = true;
GUILayout.Label("<size=30>Some <color=yellow>RICH</color> text</size>",style);
````
