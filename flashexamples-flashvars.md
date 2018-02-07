Example: Flash Vars
===================


If you wish to obtain parameters from FlashVars, you have to create a LoaderInfo object in Actionscript and access its property "parameters", which is an object that contains name-value pairs representing the parameters provided to the loaded SWF file.

Create a folder inside Assets called "ActionScript". Now create an empty textfile called "FlashVars.as" inside this folder, and insert the following code.

###ActionScript 3


````
package {
	import com.unity.UnityNative; 
	import flash.display.LoaderInfo;
	 	
	public class FlashVars {
		public static function Join(delimiter: String): String {
			var parameters: Object = LoaderInfo(UnityNative.stage.loaderInfo).parameters;
			var text: String = "";
			for (var key: String in parameters)
				text += (text.length ? delimiter : "") + key + "=" + parameters[key];
			return text;
		}
	}
}

````

This Actionscript function will obtain the FlashVars and concatenate the name-value pairs into a string that can be passed to Unity. Call the function and parse the returned list of parameters in a table.

###C\#


````
string parameters;
Dictionary<string,string> paramTable;
	
parameters = UnityEngine.Flash.ActionScript.Expression<string>("FlashVars.Join('|')");
string[] list = parameters.Split('|'); 
paramTable = new Dictionary<string, string>();
		
foreach (string parameter in list) {
	string key = parameter.Substring(0, parameter.IndexOf('='));
	string val = parameter.Substring(parameter.IndexOf('=') + 1);
	paramTable.Add(key, val);
}

````

The parameters string and parameters table are defined first. The table is a dictionary of string keys and values that will contain the FlashVars. Then the Actionscript function is called, and the name-value pairs are extracted from parameters and added to the table. Now you can read the parameters in you application. In the following example some values are shown and a given scene will be loaded as defined in FlashVars.

###C#


````
void OnGUI () {
	GUILayout.Label(" FlashVars = " + parameters);
	GUILayout.Label(" unitydebug = " + paramTable["unitydebug"]);
	GUILayout.Label(" deeptrace = " + paramTable["deeptrace"]);
	
	if (GUILayout.Button("Click here to load " + paramTable["scene"]))
		Application.LoadLevel(paramTable["scene"]);
}

````

Build the project for the first time to generate an HTML file, and edit its FlashVars for IE and other browsers (two different lines).

###HTML


````
<param name="flashvars" value="unitydebug=false&amp;deeptrace=false&amp;scene=Scene1" />


````

Finally open the HTML file in a browser to run the project.

.
