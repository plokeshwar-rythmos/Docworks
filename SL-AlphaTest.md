#ShaderLab: Legacy Alpha Testing

The alpha test is a last chance to reject a pixel from being written to the screen.

***Note:** AlphaTest commands have no effect when [fragment programs](SL-ShaderPrograms) are used; on most platforms alpha testing is done in the shader using HLSL clip() function. It is advisable to use programmable shaders these days instead of SetTexture commands.*


After the final output color has been calculated, the color can optionally have its alpha value compared to a fixed value. If the test fails, the pixel is not written to the display.


##Syntax

````
AlphaTest Off
````
Render all pixels (default) or...

````
AlphaTest comparison AlphaValue
````
Set up the alpha test to only render pixels whose alpha value is within a certain range.


###Comparison
Comparison is one of the following words:

| | |
|:---|:---|
|**Greater** |Only render pixels whose alpha is greater than _AlphaValue_. |
|**GEqual** |Only render pixels whose alpha is greater than or equal to _AlphaValue_. |
|**Less** |Only render pixels whose alpha value is less than _AlphaValue_. |
|**LEqual** |Only render pixels whose alpha value is less than or equal to from _AlphaValue_. |
|**Equal** |Only render pixels whose alpha value equals _AlphaValue_. |
|**NotEqual** |Only render pixels whose alpha value differs from _AlphaValue_. |
|**Always** |Render all pixels. This is functionally equivalent to _AlphaTest Off_. |
|**Never** |Don't render any pixels. |

###AlphaValue
A floating-point number between 0 and 1. This can also be a variable reference to a float or range property, in which case it should be written using the standard square bracket notation (_[VariableName]_).


##Details

The alpha test is important when rendering concave objects with transparent parts. The graphics card maintains a record of the depth of every pixel written to the screen. If a new pixel is further away than one already rendered, the new pixel is not written to the display. This means that even with [Blending](SL-Blend), objects will not show through.


![](../uploads/Main/sl-alphatest2.jpg) 

In this figure, the tree on the left is rendered using **AlphaTest**. Note how the pixels in it are either completely transparent or opaque. The center tree is rendered using only **Alpha Blending** - notice how transparent parts of nearby branches cover the distant leaves because of the depth buffer. The tree on the right is rendered using the last example shader - which implements a combination of blending and alpha testing to hide any artifacts.


##Examples

The simplest possible example, assign a texture with an alpha channel to it. The object will only be visible where alpha is greater than 0.5



````
Shader "Simple Alpha Test" {
	Properties {
		_MainTex ("Base (RGB) Transparency (A)", 2D) = "" {}
	}
	SubShader {
		Pass {
			// Only render pixels with an alpha larger than 50%
			AlphaTest Greater 0.5
			SetTexture [_MainTex] { combine texture }
		}
	}
}
````

This is not much good by itself. Let us add some lighting and make the cutoff value tweakable:


````
Shader "Cutoff Alpha" {
	Properties {
		_MainTex ("Base (RGB) Transparency (A)", 2D) = "" {}
		_Cutoff ("Alpha cutoff", Range (0,1)) = 0.5
	}
	SubShader {
		Pass {
			// Use the Cutoff parameter defined above to determine
			// what to render.
			AlphaTest Greater [_Cutoff]
			Material {
				Diffuse (1,1,1,1)
				Ambient (1,1,1,1)
			}
			Lighting On
			SetTexture [_MainTex] { combine texture * primary }
		}
	}
}
````


When rendering plants and trees, many games have the hard edges typical of alpha testing. A way around that is to render the object twice. In the first pass, we use alpha testing to only render pixels that are more than 50% opaque. In the second pass, we alpha-blend the graphic in the parts that were cut away, without recording the depth of the pixel. We might get a bit of confusion as further away branches overwrite the nearby ones, but in practice, that is hard to see as leaves have a lot of visual detail in them.




````
Shader "Vegetation" {
	Properties {
		_Color ("Main Color", Color) = (.5, .5, .5, .5)
		_MainTex ("Base (RGB) Alpha (A)", 2D) = "white" {}
		_Cutoff ("Base Alpha cutoff", Range (0,.9)) = .5
	}
	SubShader {
		// Set up basic lighting
		Material {
			Diffuse [_Color]
			Ambient [_Color]
		}
		Lighting On

		// Render both front and back facing polygons.
		Cull Off

		// first pass:
		// render any pixels that are more than [_Cutoff] opaque
		Pass {
			AlphaTest Greater [_Cutoff]
			SetTexture [_MainTex] {
				combine texture * primary, texture
			}
		}

		// Second pass:
		// render in the semitransparent details.
		Pass {
			// Dont write to the depth buffer
			ZWrite off
			// Don't write pixels we have already written.
			ZTest Less
			// Only render pixels less or equal to the value
			AlphaTest LEqual [_Cutoff]

			// Set up alpha blending
			Blend SrcAlpha OneMinusSrcAlpha

			SetTexture [_MainTex] {
				combine texture * primary, texture
			}
		}
	}
}
````

Note that we have some setup inside the SubShader, rather than in the individual passes. Any state set in the SubShader is inherited as defaults in passes inside it.
