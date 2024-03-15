---
created: 2024-02-28T00:20
updated: 2024-03-08T23:48
tags:
  - Rendering
  - UnrealEngine
---
# 基本概念

## G-Buffer

#G-Buffer

对 G-Buffer 进行概括的一段话：

```
GBuffer: The frame rendered out in multiple different images. 

These images are then used for compositing in anything ranging from materials to lighting to fog and so forth
```

G-Buffer 的本质是一堆图像，用来保存每一帧计算的中间结果，而每帧最终输出画面需要在这堆图像的基础上做合成。

**为什么叫 G-Buffer ?**

这涉及到延迟渲染管线 Deferred Shading，延迟渲染管线包含是一种用来渲染图像的方式，与之相对的是前向渲染管线 Forward Shading。延迟渲染包含两个阶段：geometry pass 和 lighting pass。

- Geometry Pass
	这是延迟渲染管线进行的第一步骤，该阶段将场景中所有物体的几何信息渲染到纹理图像上，几何信息包括：顶点位置、法线方向、颜色、高光等，不同类型的信息分别存储在几张纹理图像上，这些纹理图像集合称之为 G-Buffer，顾名思义就是 Geometry Buffer。
- Lighting Pass
	Lighing Pass，即光照阶段，是延迟渲染管线进行的第二步骤。有了前步骤输出的 G-Buffer，此阶段会从 G-Buffer 中拿到几何数据与光源一起做计算，得到最终渲染结果。

下面是延迟渲染管线两阶段的示意图：
![[Pasted image 20240229015353.png|550]]

上文可知，G-Buffer 用来存储延迟渲染管线的中间输出结果，主要是各种几何、材质信息，据我观察 G-Buffer 具体用哪种布局来存储信息并没有明确的规定，不同的应用、引擎的布局实现不一样，开发者也可以自行定义布局。

这是 UE 实现的 G-Buffer 布局：

![[UE 渲染基础-20240228.png|550]]

参考：
1. [第四篇：光栅化，遮蔽和GBuffer - 知乎](https://zhuanlan.zhihu.com/p/674943090)
2. [Site Unreachable](https://learnopengl.com/Advanced-Lighting/Deferred-Shading)
