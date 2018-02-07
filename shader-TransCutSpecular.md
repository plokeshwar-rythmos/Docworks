Transparent Cutout Specular
===========================

**Note.** Unity 5 introduced the [Standard Shader](shader-StandardShader) which replaces this shader.

![](../uploads/Shaders/Shader-TransCutoutSpec.png) 

One consideration for this shader is that the Base texture's alpha channel defines both the Transparent areas as well as the Specular Map.

<!-- include shader-TransCutFamilyImport -->

<!-- include shader-SpecularSubsetImport -->

Performance
-----------


Generally, this shader is moderately expensive to render. For more details, please view the [Shader Peformance page](shader-Performance).
