---
created: 2023-11-29T11:22:00
updated: 2024-02-29T19:10
title: Camera - UE 中的相机设计及项目侧实践
tags:
  - UnrealEngine
  - Gameplay
---
# UE 相机基础

用户在体验游戏时，直观上是和游戏窗口、键盘进行交互的。窗口为用户展示游戏画面，窗口里的角色代表玩家的控制对象，键盘输入角色控制指令。这些是相机系统的基本组成要素，UE 的相机系统也是围绕着这些基本组成要素设计的。

把影响用户看到画面的因素总结下，包括下面三类：

* Character: 由玩家控制的角色。用户需要能够看到自己控制的角色，所以 Character 的位置会影响用户看到的范围。
* Controller: 控制器，功能由很多，例如: 处理外界输入、控制角色行为、控制窗口可见范围。此篇文章我们只关注 Controller 对窗口可见范围的影响。
* Camera View: 相机视角/摄像机/窗口，真正最终执行渲染的摄像机，即用户实际可见的窗口。大多数情况下由 Controller 决定其能渲染哪些东西。

接下来对上述三点相机系统基本组成要素依次讨论。
![[pic01.png|500]]
## 控制器 (Controller)

关于 controller，只需要关注其内部的 `ControlRotation` 成员。
```
class AController {
    /** Controller 的控制方向 */
    FRotator ControlRotation;
}
```

`ControlRotation` 是一个旋转变量类型，他有两个作用:
1. 控制相机的朝向。Camera View 应该渲染哪个方向的场景？这由 `ControlRotation` 决定。

![[pic02.png|500]]

2. 控制角色的移动方向。角色可以前后左右移动，那么哪个方向是"前"方向呢？那就是`ControlRotation`，它将前后左右四个方向向量从局部坐标系转化到了世界坐标系空间。

![[pic03.png|500]]
## 角色 Character

Character 是用户直观控制的角色，在 UE 的相机系统设计中，总是跟随着角色移动，角色的位置决定着相机的位置。

## 相机视角 Camera View
Camera View 就是相机视角(可以理解为摄像机、渲染范围)，从上文可以得到结论，Controller 决定相机的拍摄朝向，Character 决定了相机位置。

但上述结论只是一种简单概括，实际情况会复杂点。例如我们不希望相机的位置和 Character 相等，而是期望相机放在角色身后，也就是第三人称视角，这种情况下如何设计相机位置呢？

### 第三人称视角下的相机控制

这一节要解决三个问题：
1. Controller 和 Character 朝向；
2. Character 移动方向如何决定；
3. Camera View 的朝向、位置如何计算；

如果是第一人称视角下，上述三个问题很好解答，看到许多第一视角的 FPS 游戏就知道了：
* Controller 的方向 (Control Rotation) 就是角色的朝向；
* Character 的移动方向依旧按照 Controller 的方向为正前方向。由于角色的朝向也是 Control Rotation 方向，所以在向左/向右移动时，角色的表现就是螃蟹步。
* Camera View 的朝向严格按照 Control Rotation 的朝向，位置就处于角色的位置。

在第三人称模式下，答案就有所不同了。

1. Character 的朝向
2. Character 的移动表现
3. Camera View 的朝向与位置

---

# 参考文献
- [《Exploring in UE4》摄像机系统解析](https://zhuanlan.zhihu.com/p/34897458)
---

## Controller 朝向
Pawn 的朝向和 Controller 的指向(ControlRotation) 有关系，先明确 Controller 的指向如何确定：

(1) AI Controller
Pawn 的正面朝向会跟随 AI Controller 的指向，而 AI Controller 的指向有三种方式确定：
* 如果设置了 Controller 聚焦到某个物体 Actor/Object/Position 等，Controller 指向将永远指向该物体
* 如果设置了 bSetControlRotationFromPawnOrientation 选项，Controller 的朝向由 Pawn 确定
* 否则，AI Controller 的朝向将不变

上述三种方式相互互斥.

(2) Player Controller
因为 Player Controller 控制的是玩家操控角色，它的朝向直接影响玩家看到的画面，所以影响 Player Controller 指向的因素主要是相机.
* 当 Player Controller 开始处理角色时，朝向由 Pawn 的朝向决定
* PlayerCameraManager 负责处理类似于震屏之类的效果，这类效果可通过相机抖动来实现，因此会每帧更新 PlayerController 的旋转矩阵。
* 鼠标移动产生的角度增量，将用于修改 Player Controller 朝向。

## Pawn 的朝向
Pawn 的朝向有三个 bool 选项影响：bUseControllerRotationPitch、bUseControllerRotationYaw 和 bUseControllerRotationRoll，分别用于控制 Pawn 的 Pitch、Yaw 和 Roll，代表是否由 Controller 控制 Pawn 的朝向。当任意一选项为 true 时，Pawn 的对应朝向分量将和 Controller 保持一致。

不同应用场景设置不一样，由 AI 控制的 Pawn 会将 bUseControllerRotationYaw 设置为 true，允许 AI Controller 控制 Pawn 的朝向；由玩家控制的 Pawn，如果是第三视角，则三个选项全置为 false，这将允许用户操控 Pawn 往任意方向移动而 Controller 和相机的朝向保持不变。

## 相机朝向

相机朝向由 UCameraComponent 决定。每帧相机视角会调用 UCameraComponent::GetCameraView() 函数获取相机 location、rotation 和 FOV。相机位置 location 和 rotation 又是和 UCameraComponent 紧密相关的，在 UE 中相机位置、旋转通过弹簧臂 USpringArmComponent 计算得到。SpringArm 控制相机到控制的角色（Pawn）之间的距离，顾名思义弹簧臂控制的距离是动态了，这通常用于相机避障。

对于相机旋转，则由上文提到的 Controller 控制。Controller 内部有一个 ControlRotation 矩阵来决定相机旋转角度。当我们鼠标四周移动时，修改的就是这个 ControlRotation。
