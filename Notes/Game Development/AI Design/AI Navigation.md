---
created: 2024-03-04T14:19
updated: 2024-03-04T17:25
---

## PathFollowingComponent

和 NavigationSystem、NavMovementComponent 配合，获取 Owner 的 Agent Size 数据以找到一条移动到目标的路径。路径是由一段段直线 (Segment) 连接而成，PathFollowingComponent 将每一段 Segment 方向的加速度传给 MovementComponent，让 MovementComponent 来执行 (RequestPathMove or RequestDirectMove) 移动行为。

参考：
1. [UE4：AI‘s MoveTo——代码分析\_ue moveto-CSDN博客](https://blog.csdn.net/jk_chen_acmer/article/details/120130031)