---
layout: post
title:  "last of us light tech"
date:   2020-09-20 16:13:54 +0800
categories: rendering
---
# 最后生还者1


Lightmap存在接缝漏光/相差较大的问题问题

![Lightmap Seams](/assets/images/TLOS/gen1-lightmapSeams.png)
使用最小方差去使Error function的值最小，输入值为每个图素的值

使用的Eigen 别去自己实现solver
![Minimun Square](/assets/images/TLOS/gen1-lmErrorFun.png) 

想要避免过平的间接光，当时已有以下几种方案
- half life2 basics
- sh lightmap
- ambient+lightmap per texel

最后选择了环境光+方向的方案(rgb,dir,rgb)
美术可以手调，使用烘焙工具生成了SH lightmap再生成最终结果

C_Indir = C_Ambient + Dot(N,C_IndirLDir)*C_IndirLCol

![Indirect No Shadow]

![Indirect Shadow]
  
![Direction Extraction]

需要一个可见性函数来表达受阴影程度。使用球体（允许非等比缩放来拟合胶囊体）来近似遮蔽物。

对于环境光项，可以把它看成一个cos weight函数。
对于方向光，使用cone trace。
![Shadow Approximiate]

环境光项是closed form,可以直接算。
方向光直接算有点麻烦，是用的蒙特卡洛算了以后烘焙到LUT上
phi:方向光和与遮蔽体形成的圆锥的主轴的夹角。
theta:被遮蔽的角度

![shadow lut]
![shadow result]
在spu上完成计算。渲染主导光到offscreen rt上
将屏幕分成小tile, 每个tile算出可能潜在相交的occluder





http://miciwan.com/SIGGRAPH2013/Lighting%20Technology%20of%20The%20Last%20Of%20Us.pdf






![Ambient Plus dir]


 
# 最后生还者2

