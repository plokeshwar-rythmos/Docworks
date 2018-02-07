Reflective Properties
---------------------

**Note.** Unity 5 introduced the [Standard Shader](shader-StandardShader) which replaces this shader.

This shader will simulate reflective surfaces such as cars, metal objects etc. It requires an environment Cubemap which will define what exactly is reflected. The main texture's alpha channel defines the strength of reflection on the object's surface. Any scene lights will add illumination on top of what is reflected.
