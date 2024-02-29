---
created: 2024-02-26T21:29
updated: 2024-02-29T20:01
tags:
  - Gameplay
  - AI
---
## AI 跳跃动作

AI 在 Navmesh 上移动时，如果遇到高度方向的阻挡，AI 就会停下来无法前景，典型的比如阶梯。此时需要 AI 执行跳跃动作后继续移动。

![[Pasted image 20240226195215.png]]

**方案一**

调整 Agent Size，使 NavMesh 将有高度差的位置也，然后让 AI 移动时不停的发射射线检查前方是否有高度阻挡，如果有则执行跳跃动作。

参考：[UE4 Jumping AI - Follower Bot Tutorial - YouTube](https://www.youtube.com/watch?v=M4WVRdbh_VM)

**方案二**

调用 `Character::LaunchCharacter()` 函数，设定好速度 v，将 Character 以速度 v 按弧形的方向发射出去。

![[Pasted image 20240226204010.png]]

该方案用到了 Navlink Proxy 来连接两个分割的 NavMesh。当 AI 要跨 NavMesh 移动时，会尝试从 Navlink Proxy 的起点移动到终点，但默认情况下 Navlink Proxy 并没有告诉 AI 要如何移动，所以遇到有高度的墙壁时 AI 会堵死。

为了避免 AI 堵死，Navlink Proxy 需要开启 Smart Link 模式，这种模式下 AI 经过 Navlink Proxy 会触发各种事件，例如给 Navlink Proxy 的起点绑定一个 `LaunchCharacter()` 函数，使 AI 到达起点时被发射到终点，间接的实现了 AI 的在不同高度、不同 NavMesh 之间的跳跃功能。

参考：
1. [Smart Enemy AI | (Part 10: AI Jumping / Nav Link) | Tutorial in Unreal Engine 5 (UE5) - YouTube](https://www.youtube.com/watch?v=G4GHa-zmQR8)
2. [NavLink Proxy And Smart Links; UNREAL ENGINE - YouTube](https://www.youtube.com/watch?v=iu7cjp1Gg7U)

最终采用方案二。项目中设计 NPC 的随机行为，用到了 UE 提供的 Smart Object 插件，需要在 NavLink Proxy 的基础上二次开发，将 NavLink Proxy 与 Smart Object 集成。

## AI 攀爬动作

应用场景与 AI 跳跃一样，不同的是 AI 需要采用攀爬动作到墙上。

**让动画适配高度 - Motion Warping**

参考：[UE5 Motion Warping翻越实践 - 知乎](https://zhuanlan.zhihu.com/p/466538055)


