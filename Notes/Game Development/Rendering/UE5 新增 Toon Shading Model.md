---
created: 2024-02-27T22:20
updated: 2024-02-28T03:48
tags:
  - UnrealEngine
  - Rendering
  - ShadingModel
---
按以前学过 PBRT 的经验，所谓 Shading Model 应该就是双向反射 (BRDF) \散射分布函数 (BSDF)，用来反映不同材质的入射与出射能量之间的关系，此篇笔记对在 UE 中新增 Toon Shading Model 的过程做记录。


参考：
1. [UE5新增Toon管线实践 - 知乎](https://zhuanlan.zhihu.com/p/647312365)
2. [UE5自定义着色模型 Unreal Engine 5 custom Shading Model - 知乎](https://zhuanlan.zhihu.com/p/404857208)
3. [虚幻4渲染编程(材质编辑器篇)【第二卷：自定义光照模型】 - 知乎](https://zhuanlan.zhihu.com/p/36840778)