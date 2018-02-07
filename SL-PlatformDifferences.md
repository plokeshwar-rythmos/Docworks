#Platform-specific rendering differences

Unity runs on various graphics library platforms: [Open GL](https://www.opengl.org), [Direct3D](https://msdn.microsoft.com/en-us/library/windows/desktop/hh309466.aspx), [Metal](https://developer.apple.com/metal/), and games consoles. In some cases, there are differences in how graphics rendering behaves between the platforms and Shader language semantics. Most of the time the Unity Editor hides the differences, but there are some situations where the Editor cannot do this for you. In these situations, you need to ensure that you negate differences between the platforms. These situations, and the actions you need to take if they occur, are listed below.

##Render Texture coordinates
Vertical Texture coordinate conventions differ between two types of platforms: Direct3D-like and OpenGL-like.


* **Direct3D-like**: The coordinate is 0 at the top and increases downward. This applies to Direct3D, Metal and consoles.
* **OpenGL-like**: The coordinate is 0 at the bottom and increases upward. This applies to OpenGL and OpenGL ES.

This difference tends not to have any effect on your project, other than when rendering into a [Render Texture](class-RenderTexture). When rendering into a Texture on a Direct3D-like platform, Unity internally flips rendering upside down. This makes the conventions match between platforms, with the OpenGL-like platform convention the standard. 

Image Effects and rendering in UV space are two common cases in the Shaders where you need to take action to ensure that the different coordinate conventions do not create problems in your project.

###Image Effects
When you use [Image Effects](PostProcessingWritingEffects) and anti-aliasing, the resulting source Texture for an Image Effect is not flipped to match the OpenGL-like platform convention. In this case, Unity renders to the screen to get anti-aliasing and then resolves rendering into a Render Texture for further processing with an Image Effect.

If your Image Effect is a simple one that processes one Render Texture at a time, [Graphics.Blit](ScriptRef:Graphics.Blit.html) deals with the inconsistent coordinates. However, if you’re processing more than one [Render Texture](class-RenderTexture) together in your [Image Effect](PostProcessingWritingEffects), the Render Textures are likely to come out at different vertical orientations in Direct3D-like platforms and when you use anti-aliasing. To standardise the coordinates, you need to manually “flip” the screen Texture upside down in your Vertex Shader so that it matches the OpenGL-like coordinate standard.

The following code sample demonstrates how to do this:

```
// Flip sampling of the Texture: 
// The main Texture
// texel size will have negative Y).

#if UNITY_UV_STARTS_AT_TOP
if (_MainTex_TexelSize.y < 0)
        uv.y = 1-uv.y;
#endif

```

Refer to the Edge Detection Scene in Unity's Shader Replacement sample project (see [Unity's Learn resources](http://unity3d.com/support/resources/example-projects/shader-replacement))  for a more detailed example of this. Edge detection in this project uses both the screen Texture and the Camera’s [Depth+Normals texture](SL-CameraDepthTexture).

A similar situation occurs with [GrabPass](SL-GrabPass). The resulting render Texture might not actually be turned upside down on Direct3D-like (non-OpenGL-like) platforms. If your Shader code samples GrabPass Textures, use the `ComputeGrabScreenPos` function from the [UnityCG include](SL-BuiltinFunctions) file.

###Rendering in UV space
When rendering in Texture coordinate (UV) space for special effects or tools, you might need to adjust your Shaders so that rendering is consistent between Direct3D-like and OpenGL-like systems. You also might need to adjust your rendering between rendering into the screen and rendering into a Texture. Adjust these by flipping the Direct3D-like projection upside down so its coordinates match the OpenGL-like projection coordinates.

The [built-in variable](SL-UnityShaderVariables) `ProjectionParams.x` contains a `+1` or `–1` value. `-1` indicates a projection has been flipped upside down to match OpenGL-like projection coordinates, while `+1` indicates it hasn’t been flipped. 
You can check this value in your Shaders and then perform different actions. The example below checks if a projection has been flipped and, if so, flips and then returns the UV coordinates to match.

```
float4 vert(float2 uv : TEXCOORD0) : SV_POSITION
{
    float4 pos;
    pos.xy = uv;
    // This example is rendering with upside-down flipped projection,
    // so flip the vertical UV coordinate too
    if (_ProjectionParams.x < 0)
        pos.y = 1 - pos.y;
    pos.z = 0;
    pos.w = 1;
    return pos;
}
```

##Clip space coordinates
Similar to Texture coordinates, the clip space coordinates (also known as post-projection space coordinates) differ between Direct3D-like and OpenGL-like platforms:

* **Direct3D-like**: The clip space depth goes from 0.0 at the near plane to +1.0 at the far plane. This applies to Direct3D, Metal and consoles.

* **OpenGL-like**: The clip space depth goes from –1.0 at the near plane to +1.0 at the far plane. This applies to OpenGL and OpenGL ES.

Inside Shader code, you can use the `UNITY_NEAR_CLIP_VALUE` [built-in macro](SL-BuiltinMacros) to get the near plane value based on the platform.

Inside script code, use [GL.GetGPUProjectionMatrix](ScriptRef:GL.GetGPUProjectionMatrix.html) to convert from Unity’s coordinate system (which follows OpenGL-like conventions) to Direct3D-like coordinates if that is what the platform expects.

##Precision of Shader computations
To avoid precision issues, make sure that you test your Shaders on the target platforms. The GPUs in mobile devices and PCs differ in how they treat floating point types. PC GPUs treat all floating point types (float, half and fixed) as the same - they do all calculations using full 32-bit precision, while many mobile device GPUs do not do this.

See documentation on [data types and precision](SL-DataTypesAndPrecision) for details.

##Const declarations in Shaders
Use of `const` differs between Microsoft HSL (see <a href="http://msdn.microsoft.com">msdn.microsoft.com</a>) and OpenGL’s GLSL (see [Wikipedia](https://en.wikipedia.org/wiki/OpenGL_Shading_Language)) Shader language.

* Microsoft’s HLSL `const` has much the same meaning as it does in C# and C++ in that the variable declared is read-only within its scope but can be initialized in any way.

* OpenGL’s  GLSL  `const` means that the variable is effectively a compile time constant, and so it must be initialized with compile time constraints (either literal values or calculations on other `const`s).

It is best to follow the OpenGL’s GLSL semantics and only declare a variable as `const` when it is truly invariant. Avoid initializing a `const` variable with some other mutable values (for example, as a local variable in a function). This also works in Microsoft’s HLSL, so using `const` in this way avoids confusing errors on some platforms.

##Semantics used by Shaders
To get Shaders working on all platforms, some Shader values should use these semantics:

* __Vertex Shader output (clip space) position__: `SV_POSITION`. Sometimes Shaders use POSITION semantics to get Shaders working on all platforms. Note that this does not not work on Sony PS4 or with tessellation.

* __Fragment Shader output color__: `SV_Target`. Sometimes Shaders use `COLOR` or `COLOR0` to get Shaders working on all platforms. Note that this does not work on Sony PS4.

When rendering Meshes as Points, output `PSIZE` semantics from the vertex Shader (for example, set it to 1). Some platforms, such as OpenGL ES or Metal, treat point size as “undefined” when it’s not written to from the Shader.

See documentation on [Shader semantics](SL-ShaderSemantics) for more details.

##Direct3D Shader compiler syntax
Direct3D platforms use Microsoft’s [HLSL Shader compiler](SL-ShadingLanguage). The HLSL compiler is stricter than other compilers about various subtle Shader errors. For example, it doesn’t accept function output values that aren’t initialized properly.

The most common situations that you might run into using this are:

* A [Surface Shader](SL-SurfaceShaders) vertex modifier that has an `out` parameter. Initialize the output like this:


  ```
  void vert (inout appdata_full v, out Input o) 
      {
        **UNITY_INITIALIZE_OUTPUT(Input,o);**
        // ...
      }
  ```

* Partially initialized values. For example, a function returns `float4` but the code only sets the `.xyz` values of it. Set all values or change to `float3` if you only need three values.

* Using `tex2D` in the Vertex Shader. This is not valid, because UV derivatives don’t exist in the vertex Shader. You need to sample an explicit mip level instead; for example, use `tex2Dlod` (`tex, float4(uv,0,0)`). You also need to add `#pragma target 3.0` as `tex2Dlod` is a Shader model 3.0 feature.

##DirectX 11 (DX11) HLSL syntax in Shaders 

Some parts of the [Surface Shader](SL-SurfaceShaders) compilation pipeline do not understand DirectX 11-specific HLSL (Microsoft’s shader language) syntax. 

If you’re using HLSL features like `StructuredBuffers`, `RWTextures` and other non-DirectX 9 syntax, wrap them in a DirectX X11-only preprocessor macro as shown in the example below.

```
#ifdef SHADER_API_D3D11
// DirectX11-specific code, for example
StructuredBuffer<float4> myColors;
RWTexture2D<float4> myRandomWriteTexture;
#endif
```

##Using Shader framebuffer fetch
Some GPUs (most notably PowerVR-based ones on iOS) allow you to do a form of programmable blending by providing current fragment color as input to the Fragment Shader (see `EXT_shader_framebuffer_fetch` on [khronos.org](https://www.khronos.org/registry/gles/extensions/EXT/EXT_shader_framebuffer_fetch.txt)).

It is possible to write Shaders in Unity that use the framebuffer fetch functionality. To do this, use the `inout` color argument when you write a Fragment Shader in either  HLSL (Microsoft’s shading language - see <a href="http://msdn.microsoft.com">msdn.microsoft.com</a>) or Cg (the shading language by Nvidia - see [nvidia.co.uk](http://www.nvidia.co.uk/)).

The example below is in Cg.

```
CGPROGRAM
// only compile Shader for platforms that can potentially
// do it (currently gles,gles3,metal)
#pragma only_renderers framebufferfetch

void frag (v2f i, inout half4 ocol : SV_Target)
{
    // ocol can be read (current framebuffer color)
    // and written into (will change color to that one)
    // ...
}   
ENDCG
```

##The Depth (Z) direction in Shaders

Depth (Z) direction differs on different Shader platforms.

**DirectX 11, DirectX 12, PS4, Xbox One, Metal: Reversed direction**

* The depth (Z) buffer is 1.0 at the near plane, decreasing to 0.0 at the far plane.

* Clip space range is [near,0] (meaning the near plane distance at the near plane, decreasing to 0.0 at the far plane).

**Other platforms: Traditional direction**

* The depth (Z) buffer value is 0.0 at the near plane and 1.0 at the far plane.

* Clip space depends on the specific platform: 
    * On Direct3D-like platforms, the range is [0,far] (meaning 0.0 at the near plane, increasing to the far plane distance at the far plane). 
    * On OpenGL-like platforms, the range is [-near,far] (meaning minus the near plane distance at the near plane, increasing to the far plane distance at the far plane).

Note that reversed direction depth (Z), combined with a floating point depth buffer, significantly improves depth buffer precision against the traditional direction. The advantages of this are less conflict for Z coordinates and better shadows, especially when using small near planes and large far planes.

So, when you use Shaders from platforms with the depth (Z) reversed:

* UNITY_REVERSED_Z is defined.
* `_CameraDepth` Texture texture range is 1 (near) to 0 (far).
* Clip space range is within “near” (near) to 0 (far).

However, the following macros and functions automatically work out any differences in depth (Z) directions:

* `Linear01Depth(float z)`
* `LinearEyeDepth(float z)`
* UNITY_CALC_FOG_FACTOR(coord)

###Fetching the depth Buffer
If you are fetching the depth (Z) buffer value manually, you might want to check the buffer direction. The following is an example of this:

```
float z = tex2D(_CameraDepthTexture, uv);
#if defined(UNITY_REVERSED_Z)
    z = 1.0f - z;
#endif
```

###Using clip space
If you are using clip space (Z) depth manually, you might also want to abstract platform differences by using the following macro:

`float clipSpaceRange01 = UNITY_Z_0_FAR_FROM_CLIPSPACE(rawClipSpace);`

**Note**: This macro does not alter clip space on OpenGL or OpenGL ES platforms, so it returns within “-near”1 (near) to far (far) on these platforms.  

###Projection matrices
[GL.GetGPUProjectionMatrix()](ScriptRef:GL.GetGPUProjectionMatrix.html) returns a z-reverted matrix if you are on a platform where the depth (Z) is reversed.
However, if you’re composing from projection matrices manually (for example, for custom shadows or depth rendering), you need to revert depth (Z) direction yourself where it applies via script. 

An example of this is below: 

```
var shadowProjection = Matrix4x4.Ortho(...); //shadow camera projection matrix
var shadowViewMat = ...     //shadow camera view matrix
var shadowSpaceMatrix = ... //from clip to shadowMap texture space
    
//'m_shadowCamera.projectionMatrix' is implicitly reversed 
//when the engine calculates device projection matrix from the camera projection
m_shadowCamera.projectionMatrix = shadowProjection; 

//'shadowProjection' is manually flipped before being concatenated to 'm_shadowMatrix'
//because it is seen as any other matrix to a Shader.
if(SystemInfo.usesReversedZBuffer) 
{
    shadowProjection[2, 0] = -shadowProjection[2, 0];
    shadowProjection[2, 1] = -shadowProjection[2, 1];
    shadowProjection[2, 2] = -shadowProjection[2, 2];
    shadowProjection[2, 3] = -shadowProjection[2, 3];
}
    m_shadowMatrix = shadowSpaceMatrix * shadowProjection * shadowViewMat;
```

###Depth (Z) bias 
Unity automatically deals with depth (Z) bias to ensure it matches Unity’s depth (Z) direction. However, if you are using a native code rendering plugin, you need to negate (reverse) depth (Z) bias in your C or C++ code.

####Tools to check for depth (Z) direction 
* Use [SystemInfo.usesReversedZBuffer](ScriptRef:SystemInfo-usesReversedZBuffer) to find out if you are on a platform using reversed depth (Z).
