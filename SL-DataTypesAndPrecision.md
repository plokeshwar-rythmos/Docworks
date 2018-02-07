# Shader data types and precision

The standard Shader language in Unity is [HLSL](SL-ShadingLanguage),
and general HLSL data types are supported. However, Unity has some
additions to the HLSL types, particularly for better support on mobile
platforms.


## Basic data types

The majority of calculations in shaders are carried out on floating-point numbers (which would be `float` in regular programming languages like C#). Several variants of floating point types are present: `float`, `half` and `fixed` (as well as vector/matrix variants of them, such as `half3` and `float4x4`). These types differ in precision (and, consequently, performance or power usage):

#### High precision: `float`

Highest precision floating point value; generally 32 bits (just like `float` from regular programming languages).

Full `float` precision is generally used for world space positions,
texture coordinates, or scalar computations involving complex functions such as trigonometry or power/exponentiation.

#### Medium precision: `half`

Medium precision floating point value; generally 16 bits (range of -60000 to +60000, with about 3 decimal digits of precision).

Half precision is useful for short vectors, directions, object space positions, high dynamic range colors.

#### Low precision: `fixed`

Lowest precision fixed point value. Generally 11 bits, with a range of -2.0 to +2.0 and 1/256th precision.

Fixed precision is useful for regular colors (as typically stored in regular textures) and performing simple operations on them.

#### Integer data types

Integers (`int` data type) are often used as loop counters or array indices. For this purpose, they generally work fine across various platforms.

Depending on the platform, integer types might not be supported by the GPU. For example, Direct3D 9 and OpenGL ES 2.0 GPUs only operate on floating point data, and simple-looking integer expressions (involving bit or logical operations) might be emulated using fairly complicated floating point math instructions.

Direct3D 11, OpenGL ES 3, Metal and other modern platforms have proper support for integer data types, so using bit shifts and bit masking works as expected.


## Composite vector/matrix types

HLSL has built-in vector and matrix types that are created from the basic types. For example, `float3` is a 3D vector with .x, .y, .z components, and `half4` is a medium precision 4D vector with .x, .y, .z, .w components. Alternatively, vectors can be indexed using .r, .g, .b, .a components, which is useful when working on colors.

Matrix types are built in a similar way; for example `float4x4` is a 4x4 transformation matrix. Note that some platforms only support square matrices, most notably OpenGL ES 2.0.


## Texture/Sampler types

Typically you declare textures in your HLSL code as follows:

```
sampler2D _MainTex;
samplerCUBE _Cubemap;
```

For mobile platforms, these translate into "low precision
samplers", i.e. the textures are expected to have low
precision data in them. If you know your texture contains
HDR colors, you might want to use half precision sampler:

```
sampler2D_half _MainTex;
samplerCUBE_half _Cubemap;
```

Or if your texture contains full float
precision data (e.g. [depth texture](SL-DepthTextures)), use a full precision sampler:

```
sampler2D_float _MainTex;
samplerCUBE_float _Cubemap;
```



## Precision, Hardware Support and Performance

One complication of `float`/`half`/`fixed` data type usage is that PC GPUs are **always** high precision. That is, for all the
PC (Windows/Mac/Linux) GPUs, it does not matter
whether you write `float`, `half` or `fixed` data types in your shaders.
They always compute everything in full 32-bit floating point
precision.

The `half` and `fixed` types only become relevant when
targeting mobile GPUs, where these types primarily exist for
power (and sometimes performance) constraints. Keep in
mind that you need to test your shaders on mobile to see
whether or not you are running into precision/numerical issues.

Even on mobile GPUs, the different precision support varies 
between GPU families. Here's an overview of how each mobile
GPU family treats each floating point type (indicated by the number
of bits used for it):


| GPU Family | float | half | fixed    |
|:---|:---|:---|:---|
|PowerVR Series 6/7     | 32 | 16     ||
|PowerVR SGX 5xx        | 32 | 16 | 11 |
|Qualcomm Adreno 4xx/3xx| 32 | 16     ||
|Qualcomm Adreno 2xx    | 32 vertex 24 fragment |||
|ARM Mali T6xx/7xx      | 32 | 16     ||
|ARM Mali 400/450       | 32 vertex 16 fragment       |||
|NVIDIA X1              | 32 | 16     ||
|NVIDIA K1              | 32         |||
|NVIDIA Tegra 3/4       | 32 |16      ||

Most modern mobile GPUs actually only support
either 32-bit numbers (used for `float` type) or 16-bit numbers
(used for both `half` and `fixed` types). Some older GPUs have different precisions for vertex shader and fragment shader computations.

Using lower precision can often be faster, either due to improved 
GPU register allocation, or due to special "fast path" execution
units for certain lower-precision math operations. Even when there's no raw performance advantage, using lower precision often uses less power on the GPU, leading to better battery life.

A general rule of thumb is to start with half precision for everything except positions and texture coordinates. Only increase precision if half precision is not enough for some parts of the computation.


#### Support for infinities, NaNs and other special floating point values

Support for special floating point values can be different depending on which (mostly mobile) GPU family you're running.

All PC GPUs that support Direct3D 10 support very well-specified IEEE 754 floating point
standard. This means that float numbers behave exactly like
they do in regular programming languages on the CPU.

Mobile GPUs can have slightly different levels of support. On some, dividing zero
by zero might result in a NaN ("not a number"); on others it might result in infinity, zero or any other unspecified value. Make sure to test your shaders on the target device to check they are supported.


## External GPU Documentation

GPU vendors have in-depth guides about the performance and
capabilities of their GPUs. See these for details:

* [ARM Mali Guide for Unity Developers](http://malideveloper.arm.com/documentation/developer-guides/arm-guide-unity-enhancing-mobile-games/)
* [Qualcomm Adreno OpenGL ES Developer Guide](https://developer.qualcomm.com/software/adreno-gpu-sdk/tools)
* [PowerVR Architecture Guides](https://community.imgtec.com/developers/powervr/documentation/)


## See Also

* [Platform Specific Rendering Differences](SL-PlatformDifferences)
* [Shader Performance Tips](SL-ShaderPerformance)
* [Shading Language](SL-ShadingLanguage)
