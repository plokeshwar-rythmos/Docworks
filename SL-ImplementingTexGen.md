# Implementing Fixed Function TexGen in Shaders

Before Unity 5, [texture properties](SL-Properties) could have options inside the
curly brace block, e.g. `TexGen CubeReflect`. These were controlling fixed function
texture coordinate generation. This functionality was removed in Unity 5.0; if you need
texgen you should write a [vertex shader](SL-ShaderPrograms) instead.

This page shows how to implement each of fixed function TexGen modes from Unity 4.


## Cubemap reflection (TexGen CubeReflect)

`TexGen CubeReflect` is typically used for simple [cubemap](class-Cubemap) reflections.
It reflects view direction along the normal in view space, and uses that as the UV
coordinate.

![](../uploads/SL/TexGenCubeReflect.png)


````
Shader "TexGen/CubeReflect" {
Properties {
    _Cube ("Cubemap", Cube) = "" { /* used to be TexGen CubeReflect */ }
}
SubShader { 
	Pass { 
        CGPROGRAM
        #pragma vertex vert
        #pragma fragment frag
        #include "UnityCG.cginc"
        
        struct v2f {
            float4 pos : SV_POSITION;
            float3 uv : TEXCOORD0;
        };

        v2f vert (float4 v : POSITION, float3 n : NORMAL)
        {
            v2f o;
            o.pos = UnityObjectToClipPos(v);

            // TexGen CubeReflect:
            // reflect view direction along the normal,
            // in view space
            float3 viewDir = normalize(ObjSpaceViewDir(v));
            o.uv = reflect(-viewDir, n);
            o.uv = mul(UNITY_MATRIX_MV, float4(o.uv,0));
            return o;
        }

        samplerCUBE _Cube;
        half4 frag (v2f i) : SV_Target
        {
            return texCUBE(_Cube, i.uv);
        }
        ENDCG 
    } 
}
}
````

## Cubemap normal (TexGen CubeNormal)

`TexGen CubeNormal` is typically used with [cubemaps](class-Cubemap) too.
It uses view space normal as the UV coordinate.

![](../uploads/SL/TexGenCubeNormal.png)

````
Shader "TexGen/CubeNormal" {
Properties {
    _Cube ("Cubemap", Cube) = "" { /* used to be TexGen CubeNormal */ }
}
SubShader { 
    Pass { 
        CGPROGRAM
        #pragma vertex vert
        #pragma fragment frag
        #include "UnityCG.cginc"
        
        struct v2f {
            float4 pos : SV_POSITION;
            float3 uv : TEXCOORD0;
        };

        v2f vert (float4 v : POSITION, float3 n : NORMAL)
        {
            v2f o;
            o.pos = UnityObjectToClipPos(v);

            // TexGen CubeNormal:
            // use view space normal of the object
            o.uv = mul((float3x3)UNITY_MATRIX_IT_MV, n);
            return o;
        }

        samplerCUBE _Cube;
        half4 frag (v2f i) : SV_Target
        {
            return texCUBE(_Cube, i.uv);
        }
        ENDCG 
    } 
}
}
````

## Object space coordinates (TexGen ObjectLinear)

`TexGen ObjectLinear` used object space vertex position as UV coordinate.

![](../uploads/SL/TexGenObjectLinear.png)

````
Shader "TexGen/ObjectLinear" {
Properties {
    _MainTex ("Texture", 2D) = "" { /* used to be TexGen ObjectLinear */ }
}
SubShader { 
    Pass { 
        CGPROGRAM
        #pragma vertex vert
        #pragma fragment frag
        #include "UnityCG.cginc"
        
        struct v2f {
            float4 pos : SV_POSITION;
            float3 uv : TEXCOORD0;
        };

        v2f vert (float4 v : POSITION)
        {
            v2f o;
            o.pos = UnityObjectToClipPos(v);

            // TexGen ObjectLinear:
            // use object space vertex position
            o.uv = v.xyz;
            return o;
        }

        sampler2D _MainTex;
        half4 frag (v2f i) : SV_Target
        {
            return tex2D(_MainTex, i.uv.xy);
        }
        ENDCG 
    } 
}
}
````

## View space coordinates (TexGen EyeLinear)

`TexGen EyeLinear` used view space vertex position as UV coordinate.

![](../uploads/SL/TexGenEyeLinear.png)

````
Shader "TexGen/EyeLinear" {
Properties {
    _MainTex ("Texture", 2D) = "" { /* used to be TexGen EyeLinear */ }
}
SubShader { 
    Pass { 
        CGPROGRAM
        #pragma vertex vert
        #pragma fragment frag
        #include "UnityCG.cginc"
        
        struct v2f {
            float4 pos : SV_POSITION;
            float3 uv : TEXCOORD0;
        };

        v2f vert (float4 v : POSITION)
        {
            v2f o;
            o.pos = UnityObjectToClipPos(v);

            // TexGen EyeLinear:
            // use view space vertex position
            o.uv = UnityObjectToViewPos(v);
            return o;
        }

        sampler2D _MainTex;
        half4 frag (v2f i) : SV_Target
        {
            return tex2D(_MainTex, i.uv.xy);
        }
        ENDCG 
    } 
}
}
````


## Spherical environment mapping (TexGen SphereMap)

`TexGen SphereMap` computes UV coordinates for spherical environment mapping.
See [OpenGL TexGen reference](http://docs.gl/gl2/glTexGen) for the formula.

![](../uploads/SL/TexGenSphereMap.png)


````
Shader "TexGen/SphereMap" {
Properties {
    _MainTex ("Texture", 2D) = "" { /* used to be TexGen SphereMap */ }
}
SubShader { 
    Pass { 
        CGPROGRAM
        #pragma vertex vert
        #pragma fragment frag
        #include "UnityCG.cginc"
        
        struct v2f {
            float4 pos : SV_POSITION;
            float2 uv : TEXCOORD0;
        };

        v2f vert (float4 v : POSITION, float3 n : NORMAL)
        {
            v2f o;
            o.pos = UnityObjectToClipPos(v);

            // TexGen SphereMap
            float3 viewDir = normalize(ObjSpaceViewDir(v));
            float3 r = reflect(-viewDir, n);
            r = mul((float3x3)UNITY_MATRIX_MV, r);
            r.z += 1;
            float m = 2 * length(r);
            o.uv = r.xy / m + 0.5;

            return o;
        }

        sampler2D _MainTex;
        half4 frag (v2f i) : SV_Target
        {
            return tex2D(_MainTex, i.uv);
        }
        ENDCG 
    } 
}
}
````
