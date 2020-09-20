---
layout: post
title:  "Shading of Source"
date:   2020-09-15 15:13:54 +0800
categories: jekyll update
---

Radiosity Normal Mapping


// https://steamcdn-a.akamaihd.net/apps/valve/2004/GDC2004_Half-Life2_Shading.pdf
lightmapColor[0] * dot(bumpBasis[0], normal) +
lightmapColor[1] * dot(bumpBasis[1], normal) +
lightmapColor[2] * dot(bumpBasis[2], normal)

Ambient Cube

// https://steamcdn-a.akamaihd.net/apps/valve/2004/GDC2004_Half-Life2_Shading.pdf
// p50
float3 AmbientLight(const float3 worldNormal)
{
    float3 nSquared = worldNormal * worldNormal; 
    // x^2 + y^2 + z^2 = 1
    // imagine doing a linear interpolation on a sphere
    // I am understanding it in the similar way to slerp
    // plz let me know if I am doing wrong
    int3 isNegative = (worldNormal < 0.0);
    float3 linearColor;
    linearColor = nSquared.x * cAmbientCube[isNegative.x] +
                  nSquared.y * cAmbientCube[isNegative.y+2] + 
                  nSquared.z * cAmbientCube[isNegative.z+4];
    return linearColor;
}

This technique could be also used for faking volumetric cloud rendering.
https://twitter.com/Vuthric/status/1286796950214307840
https://realtimevfx.com/t/smoke-lighting-and-texture-re-usability-in-skull-bones/5339