---
layout: post
title:  "VULKAN, HLSL and DXMATH"
date:   2020-11-27 1:0:00 +0800
categories: rendering
---


## Intro
For short, I am alwyas confused about the row-column major of graphics API.
Especially when it comes to vulkan, with new NDC, and HLSL which is designed for DX though it's 'native' shading language now.
So I deceide to make it clear and write down this article.

## First Attempt
After try out bunch of permutation, the following code works.

``` cpp

// DXMATH MATRIX SHOULD BE ROW MAJOR
ubo.model = Matrix::CreateRotationY(time);
ubo.view = Matrix::CreateLookAt(float3(2,2,2), float3(0,0,0), float3(0,1,0));
ubo.view = ubo.view;
ubo.proj = Matrix::CreatePerspectiveFieldOfView(3.14f / 2.0f, m_swapChainExtent.width / (float) m_swapChainExtent.height, 0.01f, 1000.0f);
ubo.proj._22 *= -1;
ubo.proj = ubo.proj;


VSOutput main(VSInput input)
{
	VSOutput output = (VSOutput)0;
	output.Color = input.Color;
    // but the multiply style is more like column major
    float4 pos =  mul(ubo.proj, mul(ubo.view, (mul(ubo.model, float4(input.Pos.xyz, 1.0)))));
    output.Pos =  pos;
	return output;
}

```

But this is a little bit wierd for me, since the matrix should be row major, while the hlsl multiply is column major style.

## Column Major and Row Major
[scratchapixel reference](https://www.scratchapixel.com/lessons/mathematics-physics-for-computer-graphics/geometry/row-major-vs-column-major-vector)

DXMATH: ROW MAJOR

GLM: COL MAJOR
```
Row Major : V = [x y z]

                  x
Col Major : V = [ y ]
                  z
```

Row Major: Left or pre-multiplication P/V * M or (P *  S * R * T)
Column Major: Right or post-multiplication M * P/V or (T * R * S * P) 

```
           a b c
[x y z] * [d e f] = [x' y' z']
           g h i

x' = x * a + y * d + z * g
y' = x * b + y * e + z * h
z' = x * c + y * f + z * i

```

```
 a b c
[d e f] = [x' y' z']
 g h i

x' = x * a + y * d + z * g
y' = x * b + y * e + z * h
z' = x * c + y * f + z * i

```




## COMPILER
From [hlsl-for-vulkan-matrices](antiagainst.github.io/post/hlsl-for-vulkan-matrices/)
When translate HLSL to SPIR_V

>### Majorness
>As said previously, majorness only matters for externally initialized matrices. This agrees with SPIR-V validation rules, which requires matrices inside shader resources to be explicitly laid out, but not the shader internal ones (inside Function or Private storage class).

>### Float matrices
>But as we need to swap the multiplication operands, we also need to flip the majorness decoration in SPIR-V to make sure matrices are initialized in the transposed manner:
>
>HLSL row_major should be translated into SPIR-V ColMajor;
>HLSL column_major should be translated into SPIR-V RowMajor.


After reading this I start to understand why this happened.
The Matrix set from cpu is row major, but when doing the calculating, the compiler transpose it to col major, thus my right multiply works.

There are two ways to fix it, adding the decorator before matrix
```
struct UBO
{
	row_major float4x4 model;
	row_major float4x4 view;
	row_major float4x4 proj;
};
cbuffer ubo : register(b0) { UBO ubo; }

VSOutput main(VSInput input)
{
	VSOutput output = (VSOutput)0;
	output.Color = input.Color;
    float4 pos =  mul(mul(mul(float4(input.Pos.xyz, 1.0), ubo.model),ubo.view),ubo.proj); // row major
    output.Pos =  pos;
	return output;
}
// https://github.com/microsoft/DirectXShaderCompiler/blob/master/docs/SPIR-V.rst
```

or using the compile cmd -Zpc/-Zpr

second somehow not work for me for now. So I decided to stick with row major in my shader as long as make it consitent with PBRT.

## Conclusion
That's it. Just some anti intuition setup from compiler.
