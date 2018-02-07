#Texture arrays

Similar to regular 2D textures ([Texture2D](class-TextureImporter) class, __sampler2D__ in shaders), cube maps ([Cubemap](class-Cubemap) class, __samplerCUBE__ in shaders), and 3D textures ([Texture3D](class-Texture3D) class, __sampler3D__ in shaders), Unity also supports 2D texture arrays.

A texture array is a collection of same size/format/flags 2D textures that look like a single object to the GPU, and can be sampled in the shader with a texture element index. They are useful for implementing custom terrain rendering systems or other special effects where you need an efficient way of accessing many textures of the same size and format. Elements of a 2D texture array are also known as slices, or layers.

##Platform Support

Texture arrays need to be supported by the underlying graphics API and the GPU. They are available on:

* Direct3D 11/12 (Windows, Xbox One)
* OpenGL Core (Mac OS X, Linux)
* Metal (iOS, Mac OS X)
* OpenGL ES 3.0 (Android, iOS, WebGL 2.0)
* PlayStation 4

Other platforms do not support texture arrays (OpenGL ES 2.0 or WebGL 1.0). Use [SystemInfo.supports2DArrayTextures](ScriptRef:SystemInfo-supports2DArrayTextures.html) to determine texture array support at runtime.

##Creating and manipulating texture arrays

As there is no texture import pipeline for texture arrays, they must be created from within your scripts. Use the [Texture2DArray](ScriptRef:Texture2DArray.html) class to create and manipulate them. Note that texture arrays can be serialized as assets, so it is possible to create and fill them with data from editor scripts.

Normally, texture arrays are used purely within GPU memory, but you can use [Graphics.CopyTexture](ScriptRef:Graphics.CopyTexture.html), [Texture2DArray.GetPixels](ScriptRef:Texture2DArray.GetPixels.html) and [Texture2DArray.SetPixels](ScriptRef:Texture2DArray.SetPixels.html) to transfer pixels to and from system memory.

##Using texture arrays as render targets
Texture array elements may also be used as render targets. Use [RenderTexture.dimension](ScriptRef:RenderTexture-dimension.html) to specify in advance whether the render target is to be a 2D texture array. The __depthSlice__ argument to [Graphics.SetRenderTarget](ScriptRef:Graphics.SetRenderTarget.html) specifies which mipmap level or cube map face to render to. On platforms that support “layered rendering” (i.e. geometry shaders), you can set the __depthSlice__ argument to -1 to set the whole texture array as a render target. You can also use a geometry shader to render into individual elements.

##Using texture arrays in shaders
Since texture arrays do not work on all platforms, shaders need to use an appropriate [compilation target](SL-ShaderCompileTargets) to access them. The minimum shader model compilation target that supports texture arrays is 3.5.

Use these [macros](SL-BuiltinMacros) to declare and sample texture arrays:

* UNITY_DECLARE_TEX2DARRAY(name) declares a texture array sampler variable inside HLSL code.
* UNITY_SAMPLE_TEX2DARRAY(name,uv) samples a texture array with a __float3__ UV; the z component of the coordinate is an array element index.
* UNITY_SAMPLE_TEX2DARRAY_LOD(name,uv,lod) samples a texture array with an explicit mipmap level.

##Examples
The following shader example samples a texture array using object space vertex positions as coordinates:

````
Shader "Example/Sample2DArrayTexture"
{
    Properties
    {
        _MyArr ("Tex", 2DArray) = "" {}
        _SliceRange ("Slices", Range(0,16)) = 6
        _UVScale ("UVScale", Float) = 1.0
    }
    SubShader
    {
        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            // to use texture arrays we need to target DX10/OpenGLES3 which
            // is shader model 3.5 minimum
            #pragma target 3.5
            
            #include "UnityCG.cginc"

            struct v2f
            {
                float3 uv : TEXCOORD0;
                float4 vertex : SV_POSITION;
            };

            float _SliceRange;
            float _UVScale;

            v2f vert (float4 vertex : POSITION)
            {
                v2f o;
                o.vertex = mul(UNITY_MATRIX_MVP, vertex);
                o.uv.xy = (vertex.xy + 0.5) * _UVScale;
                o.uv.z = (vertex.z + 0.5) * _SliceRange;
                return o;
            }
            
            UNITY_DECLARE_TEX2DARRAY(_MyArr);

            half4 frag (v2f i) : SV_Target
            {
                return UNITY_SAMPLE_TEX2DARRAY(_MyArr, i.uv);
            }
            ENDCG
        }
    }
}
````


##See Also
* [Introduction To Textures](https://msdn.microsoft.com/en-us/library/windows/desktop/ff476906.aspx#Texture2D_Resource) in Direct3D documentation.
* [Array Textures](https://www.opengl.org/wiki/Array_Texture) in OpenGL Wiki.

