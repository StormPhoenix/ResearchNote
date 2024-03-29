---
created: 2024-03-04T14:19
updated: 2024-03-18T16:40
tags:
  - AI
  - UnrealEngine
---
# Behavior Tree

[UE代码-游戏AI-行为树执行流程（待更新） - ccsu\_madoka - 博客园](https://www.cnblogs.com/whitelily/p/17100961.html)

# Navigation
## PathFollowingComponent

和 NavigationSystem、NavMovementComponent 配合，获取 Owner 的 Agent Size 数据以找到一条移动到目标的路径。路径是由一段段直线 (Segment) 连接而成，PathFollowingComponent 将每一段 Segment 方向的加速度传给 MovementComponent，让 MovementComponent 来执行 (RequestPathMove or RequestDirectMove) 移动行为。PathFollowingComponent 负责重要的寻路计算，底层调用 A* 寻路算法得到移动路径。

参考：
1. [UE4：AI‘s MoveTo——代码分析\_ue moveto-CSDN博客](https://blog.csdn.net/jk_chen_acmer/article/details/120130031)
2. [从源码深入理解Unreal的AI寻路系统\_unreal寻路-CSDN博客](https://blog.csdn.net/hacning/article/details/132146914)
