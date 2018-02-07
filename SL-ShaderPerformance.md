# Performance tips when writing shaders


## Only compute what you need


The more computations and processing your shader code needs to do, the more it will impact the performance of your game. For example, supporting color per material is nice to make a shader more flexible, but if you always leave that color set to white then useless computations are performed for each vertex or pixel rendered on screen.

The frequency of computations will also impact the performance of your game. Usually there are many more pixels rendered (and subsequently more pixel shader executions) than there are vertices (vertex shader executions), and more vertices than objects being rendered. Where possible, move computations out of the pixel shader code into the the vertex shader code, or move them out of shaders completely and set the values in a script.


## Optimized Surface Shaders


[Surface Shaders](SL-SurfaceShaders) are great for writing shaders that interact with lighting. However, their default options are tuned to cover a broad number of general cases. Tweak these for specific situations to make shaders run faster or at least be smaller:

* The `approxview` directive for shaders that use view direction (i.e. Specular) makes the view direction normalized per vertex instead of per pixel. This is approximate, but often good enough.
* The `halfasview` for Specular shader types is even faster. The half-vector (halfway between lighting direction and view vector) is computed and normalized per vertex, and the [lighting function](SL-SurfaceShaderLighting) receives the half-vector as a parameter instead of the view vector.
* `noforwardadd` makes a shader fully support one-directional light in Forward rendering only. The rest of the lights can still have an effect as per-vertex lights or spherical harmonics. This is great to make your shader smaller and make sure it always renders in one pass, even with multiple lights present.
* `noambient` disables ambient lighting and spherical harmonics lights on a shader. This can make performance slightly faster.


## Precision of computations


When writing shaders in Cg/HLSL, there are three basic number types: `float`, `half` and `fixed` (see [Data Types and Precision](SL-DataTypesAndPrecision)). 

For good performance, always use the lowest precision that is possible. This is especially important on mobile platforms like iOS and Android. Good rules of thumb are:

* For world space positions and texture coordinates, use `float` precision.
* For everything else (vectors, HDR colors, etc.), start with `half` precision. Increase only if necessary.
* For very simple operations on texture data, use `fixed` precision.

In practice, exactly which number type you should use for depends on the platform and the GPU. Generally speaking:

* All modern desktop GPUs will always compute everything in full `float` precision, so `float/half/fixed` end up being exactly the same underneath. This can make testing difficult, as it's harder to see if half/fixed precision is really enough, so always test your shaders on the target device for accurate results.
* Mobile GPUs have actual `half` precision support. This is usually faster, and uses less power to do calculations.
* `Fixed` precision is generally only useful for older mobile GPUs. Most modern GPUs (the ones that can run OpenGL ES 3 or Metal) internally treat `fixed` and `half` precision exactly the same.

See [Data Types and Precision](SL-DataTypesAndPrecision) for more details.


## Alpha Testing


The fixed-function [AlphaTest](SL-AlphaTest) - or its programmable equivalent, `clip()` - has different performance characteristics on different platforms:

* Generally you gain a small advantage when using it to remove totally transparent pixels on most platforms.
* However, on PowerVR GPUs found in iOS and some Android devices, alpha testing is resource-intensive. Do not try to use it for performance optimization on these platforms, as it causes the game to run slower than usual.


## Color Mask


On some platforms (mostly mobile GPUs found in iOS and Android devices), using [ColorMask](SL-Pass) to leave out some channels (e.g. `ColorMask RGB`) can be resource-intensive, so only use it if really necessary.
