Custom Material Editors
=======================


Sometimes you have a shader with some interesting data types that can not be nicely represented using the built in Unity material editor. Unity provides a way to override the default material inspector so that you can define your own. You can use this feature to define custom controls and data range validation.

The first part to writing a custom material editor is defining a shader that requires a [Custom Editor](SL-CustomEditor). The name you use for the custom editor is the class that will be looked up by Unity for the material editor.

To define a custom editor you extend from the MaterialEditor class and place the script below an Editor folder in the assets directory.



````
using UnityEditor;

public class CustomMaterialInspector : MaterialEditor {
	public override void OnInspectorGUI () {
		base.OnInspectorGUI ();
	}
}
````

Any shader that has a custom editor defined (**CustomEditor "CustomMaterialInspector"**) will find the editor listed above and execute the associated code. 

A real example
--------------

So we have a situation where we have a shader that can work in two modes; it renders standard diffuse lighting or it only renders the red components of the source texture. This is achieved using [shader multi compilation](SL-MultipleProgramVariants).



````
Shader "Custom/Redify" {
	Properties {
		_MainTex ("Base (RGB)", 2D) = "white" {}
	}
	SubShader {
		Tags { "RenderType"="Opaque" }
		LOD 200
		
		CGPROGRAM
		#pragma surface surf Lambert
		#pragma multi_compile REDIFY_ON REDIFY_OFF

		sampler2D _MainTex;

		struct Input {
			float2 uv_MainTex;
		};

		void surf (Input IN, inout SurfaceOutput o) {
			half4 c = tex2D (_MainTex, IN.uv_MainTex);
			o.Albedo = c.rgb;
			o.Alpha = c.a;


	#if REDIFY_ON
			o.Albedo.gb = (o.Albedo.g + o.Albedo.b) / 2.0;

	#endif
		}
		ENDCG
	} 
	FallBack "Diffuse"
	CustomEditor "CustomMaterialInspector"
}
````

As you can see the shader has two Keywords available for setting: REDIFY_ON and REDIFY_OFF. These can be changed be set on a per material basis by using the shaderKeywords property of the material. below is an editor that does this.



````
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;
using System.Linq;

public class CustomMaterialInspector : MaterialEditor {

	public override void OnInspectorGUI ()
	{
		// render the default inspector
		base.OnInspectorGUI ();

		// if we are not visible... return
		if (!isVisible)
			return;

		// get the current keywords from the material
		Material targetMat = target as Material;
		string[] keyWords = targetMat.shaderKeywords;

		// see if redify is set, then show a checkbox
		bool redify = keyWords.Contains ("REDIFY_ON");
		EditorGUI.BeginChangeCheck();
		redify = EditorGUILayout.Toggle ("Redify material", redify);
		if (EditorGUI.EndChangeCheck())
		{
			// if the checkbox is changed, reset the shader keywords
			var keywords = new List<string> { redify ? "REDIFY_ON" : "REDIFY_OFF"};
			targetMat.shaderKeywords = keywords.ToArray ();
			EditorUtility.SetDirty (targetMat);
		}
	}
}
````


How the default material editor works
-------------------------------------

The default Unity editor renders all the properties that exist in a shader to the the material editor. Below is a custom material editor that behaves similar to the Unity material editor. Use this example if you wish to manually render the default fields.



````
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;
using System.Linq;

public class CustomMatInspector : MaterialEditor {

	// this is the same as the ShaderProperty function, show here so 
	// you can see how it works
	private void ShaderPropertyImpl(Shader shader, int propertyIndex)
	{
		int i = propertyIndex;
		string label = ShaderUtil.GetPropertyDescription(shader, i);
		string propertyName = ShaderUtil.GetPropertyName(shader, i);
		switch (ShaderUtil.GetPropertyType(shader, i))
		{
			case ShaderUtil.ShaderPropertyType.Range: // float ranges
			{
				GUILayout.BeginHorizontal();
				float v2 = ShaderUtil.GetRangeLimits(shader, i, 1);
				float v3 = ShaderUtil.GetRangeLimits(shader, i, 2);
				RangeProperty(propertyName, label, v2, v3);
				GUILayout.EndHorizontal();

				break;
			}
			case ShaderUtil.ShaderPropertyType.Float: // floats
			{
				FloatProperty(propertyName, label);
				break;
			}
			case ShaderUtil.ShaderPropertyType.Color: // colors
			{
				ColorProperty(propertyName, label);
				break;
			}
			case ShaderUtil.ShaderPropertyType.TexEnv: // textures
			{
				ShaderUtil.ShaderPropertyTexDim desiredTexdim = ShaderUtil.GetTexDim(shader, i);
				TextureProperty(propertyName, label, desiredTexdim);

				GUILayout.Space(6);
				break;
			}
			case ShaderUtil.ShaderPropertyType.Vector: // vectors
			{
				VectorProperty(propertyName, label);
				break;
			}
			default:
			{
				GUILayout.Label("ARGH" + label + " : " + ShaderUtil.GetPropertyType(shader, i));
				break;
			}
		}
	}

	public override void OnInspectorGUI ()
	{
		serializedObject.Update ();
		var theShader = serializedObject.FindProperty ("m_Shader");	
		if (isVisible && !theShader.hasMultipleDifferentValues && theShader.objectReferenceValue != null)
		{
			float controlSize = 64;

			EditorGUIUtility.LookLikeControls(Screen.width - controlSize - 20);

			EditorGUI.BeginChangeCheck();
			Shader shader = theShader.objectReferenceValue as Shader;

			for (int i = 0; i < ShaderUtil.GetPropertyCount(shader); i++)
			{
				ShaderPropertyImpl(shader, i);
			}

			if (EditorGUI.EndChangeCheck())
				PropertiesChanged ();
		}
	}
}
````
