#Making multiple shader program variants


Often it is convenient to keep most of a piece of shader code fixed but also allow slightly different shader "variants" to be produced. This is commonly called "mega shaders" or "uber shaders", and is achieved by compiling the shader code multiple times with different preprocessor directives for each case.

In Unity this can be achieved by adding a `#pragma multi_compile` or `#pragma shader_feature` directive to a [shader snippet](SL-ShaderPrograms). This works in [surface shaders](SL-SurfaceShaders) too.

At runtime, the appropriate shader variant is picked up from the Material keywords (Material.EnableKeyword and DisableKeyword) or global shader keywords (Shader.EnableKeyword and DisableKeyword).


## How multi\_compile works

A directive like:

````
#pragma multi_compile FANCY_STUFF_OFF FANCY_STUFF_ON
````

Will produce two shader variants, one with `FANCY_STUFF_OFF` defined, and another with `FANCY_STUFF_ON`. At runtime, one of them will be activated based on the Material or global shader keywords. If neither of these two keywords are enabled then the first one ("off") will be used.

There can be more than two keywords on a multi\_compile line, for example this will produce four shader variants:

````
#pragma multi_compile SIMPLE_SHADING BETTER_SHADING GOOD_SHADING BEST_SHADING
````

When any of the names are all underscores, then a shader variant will be produced, with no preprocessor macro defined. This is commonly used for shaders features, to avoid using up two keywords (see notes on keywork limit below). For example, the directive below will produce two shader variants; first one with nothing defined, and second one with `FOO_ON` defined:

````
#pragma multi_compile __ FOO_ON
````


## Difference between shader\_feature and multi\_compile

`#pragma shader_feature` is very similar to `#pragma multi_compile`, the only difference is that unused variants of shader\_feature shaders will not be included into game build. So shader\_feature makes most sense for keywords that will be set on the materials, while multi\_compile for keywords that will be set from code globally.

Additionally, it has a shorthand notation with just one keyword:

````
#pragma shader_feature FANCY_STUFF
````

Which is just a shortcut for `#pragma shader_feature _ FANCY_STUFF`, i.e. it expands into two shader variants (first one without the define; second one with it).



##Combining several multi\_compile lines

Several multi_compile lines can be provided, and the resulting shader will be compiled for all possible combinations of the lines:

````
#pragma multi_compile A B C
#pragma multi_compile D E
````

This would produce three variants for first line, and two for the second line, or in total six shader variants (A+D, B+D, C+D, A+E, B+E, C+E).

It's easiest to think of each multi\_compile line as controlling a single shader "feature". Keep in mind that the total number of shader variants grows really fast this way. For example, ten multi\_compile "features" with two options each produces 1024 shader variants in total!


##Keyword limit

When using Shader variants, remember that there is a limit of 256 keywords in Unity, and around 60 of them are used internally (therefore lowering the available limit). Also, the keywords are enabled globally throughout a particular Unity project, so be careful not to exceed the limit when multiple keywords are defined in several different Shaders.


## Built-in multi\_compile shortcuts

There are several "shortcut" notations for compiling multiple shader variants; they are mostly to deal with different light, shadow and lightmap types in Unity. See [rendering pipeline](SL-RenderPipeline) for details.

* `multi_compile_fwdbase` compiles all variants needed by `ForwardBase` (forward rendering base) pass type. The variants deal with different lightmap types and main directional light having shadows on or off.
* `multi_compile_fwdadd` compiles variants for `ForwardAdd` (forward rendering additive) pass type. This compiles variants to handle directional, spot or point light types, and their variants with cookie textures.
* `multi_compile_fwdadd_fullshadows` - same as above, but also includes ability for the lights to have realtime shadows.
* `multi_compile_fog` expands to several variants to handle different fog types (off/linear/exp/exp2).


Most of the built-in shortcuts result in many shader variants. It is possible to skip compiling some of them if you know they are not needed, by using `#pragma skip_variants`. For example:

````
#pragma multi_compile_fwdadd
// will make all variants containing
// "POINT" or "POINT_COOKIE" be skipped
#pragma skip_variants POINT POINT_COOKIE
````

## Shader Hardware Variants

One common reason for using shader variants is to create fallbacks or simplified shaders that can run efficiently on both high and low end hardware within a single target platform - such as OpenGL ES. To provide a specially optimised set of variants for different levels of hardware capability, you can use shader hardware variants.

To enable the generation of shader hardware variants, add `#pragma hardware_tier_variants renderer`, where `renderer` is one of the available renderering platforms for [shader program pragmas](SL-ShaderPrograms). With this `#pragma` 3 shader variants will be generated for each shader, regardless of any other keywords. Each variant will have one of the following defined:

```
UNITY_HARDWARE_TIER1
UNITY_HARDWARE_TIER2
UNITY_HARDWARE_TIER3
```

You can use these to write conditional fallbacks or extra features for lower or higher end. In the editor you can test any of the tiers by using the Graphics Emulation menu, which allows you to change between each of the tiers.

To help keep the impact of these variants as small as possible, only one set of shaders is ever loaded in the player. In addition, any shaders which end up identical - for example if you only write a specialised version for TIER1 and all others are the same - will not take up any extra space on disk.

At load time Unity will examine the GPU that it is using and auto-detect a tier value; it will default to the highest tier if the GPU is not auto-detected. You can override this tier value by setting `Shader.globalShaderHardwareTier`, but this must be done before any shaders you want to vary are loaded. Once the shaders are loaded they will have selected their set of variants and this value will have no effect. A good place to set this would be in a pre-load scene before you load your main scene.

Note that these shader hardware tiers are not related to the quality settings of the player, they are purely detected from the relative capability of the GPU the player is running on.

## Platform Shader Settings

Apart from tweaking your shader code for different hardware tiers, you might want to tweak unity internal defines (e.g. you might want to force cascaded shadowmaps on mobiles). You can find details on this in the  [UnityEditor.Rendering.PlatformShaderSettings](ScriptRef:Rendering.PlatformShaderSettings.html) documentation, which provides a list of currently supported features for overriding per-tier.
Use [UnityEditor.Rendering.EditorGraphicsSettings.SetShaderSettingsForPlatform](ScriptRef:Rendering.EditorGraphicsSettings.SetShaderSettingsForPlatform.html) to tweak Platform Shader Settings per-platform per-tier.

Please note that if `PlatformShaderSetting`s set to different tiers are not identical, then tier variants will be generated for the shader even if `#pragma hardware_tier_variants` is missing.

### See Also

* [Optimizing Shader Load Time](OptimizingShaderLoadTime).
