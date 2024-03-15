---
created: 2024-02-26T21:29
updated: 2024-03-09T00:35
tags:
  - Gameplay
  - AI
---
# AI 跳跃动作

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

---
# AI 攀爬动作

应用场景与 AI 跳跃一样，不同的是 AI 需要采用攀爬动作到墙上。

**让动画适配高度 - Motion Warping**

参考：[UE5 Motion Warping翻越实践 - 知乎](https://zhuanlan.zhihu.com/p/466538055)

---
# AI 沿路径点平滑移动

AI 在沿着路径点移动时，要求在经过路径点的时候不要停顿下来，而是平滑的移动。此处涉及到两点，是移动平滑还是路径平滑？移动平滑是指严格沿着路径点折线移动，且移动不能停；路径平滑是指移动的路径是连续可导，这就要求不能是严格按照曲线移动。

行为树上用到的 AI 移动节点调用的是 UAITask_MoveTo。UAITask_MoveTo 先用烘培好的 NavigationMesh 计算出折现路径，然后用 [[AI 原理解读#PathFollowingComponent]] 组件实现 AI 的移动，而 PathFollowingComponent 内部将移动的方向、速度计算好后又是通过 MovementComponent 来移动的。
**方案一**

使用 Splineline，设置 AI 的 Location / Rotation

**方案二**

参考 PathFollowingComponent 中调用 NavMovementComponent 的 RequestPathMove \ RequestDirectMove 使 AI 移动

**方案三**

自定义 AITask，内部多次调用 AITask_MoveTo。为了让 AI 移动时更加顺滑，需要调整 Ground Friction，让 AI 的速度计算符合加速度公式（避免 UE 中为了近似摩擦力效果的速度插值逻辑）。
其次，要覆写 CharacterMovementComponent 中的旋转插值逻辑，让 AI 的旋转用当前速度方向而不是加速度方向来插值。


参考：
[Make an Ai Follow a Spline in Unreal Engine 4 - YouTube](https://www.youtube.com/watch?v=UIF1PcmZkGA) 使用 Controller 的 MoveToLocation 使 AI 移动
[Move Objects Over a Spline - UE4/UE5 Tutorial - YouTube](https://www.youtube.com/watch?v=HYFBmx6QRfs) 调用 SetActorLocation 和 SetActorRotation 强制移动 AI 的位姿

---
## AI & Player 交互

### AI Perception 概览

要让 AI 看到角色产生反应，只要编写 `OnTargetPerceptionUpdated` 事件就好了，Perception 组件会在感受到刺激源时调用这个事件。

![[AI 不同的行为实现-20240307.png]]
- Stimulus Location: 被感知到的刺激源 Pawn 位置；
- Receiver Location: 感知主体的位置；
- Successfully Sensed: 当刺激源开始被感知为 true，脱离感知则为 false；

### 源码解读

要实现 AI 的感知行为，只要关注这个事件就好了，我们先来看看事件参数含义是什么。

![[AI 不同的行为实现-20240307.png|500]]

- Stimulus Location: 被感知到的物体位置
- Receiver Location: 感知主体的位置

### 如何打断行为树

引擎自带的 Task 节点 --- Wait、MoveTo --- 是可以随着行为树的打断而中断的，我们来看看原理是什么？

#### 行为树节点 UBT Task 的中断逻辑 - Abort Task

BTTask_BlueprintBase::AbortTask：执行中断逻辑，取消所有在 LatentActionManager 中注册的延迟行为，其次执行用户自定义的 Abort 行为。
~~~cpp
EBTNodeResult::Type UBTTask_BlueprintBase::AbortTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory)  
{  
	// 取消注册的延迟执行函数
	BlueprintNodeHelpers::AbortLatentActions(OwnerComp, *this);
	// ... ...
	// 执行中断逻辑
    ReceiveAbortAI(AIOwner, AIOwner->GetPawn());  
}
~~~

BTTask_BlueprintBase::AbortTask 是从 UBTTaskNode 基类继承而来的，这个基类里啥逻辑都没有，完全依赖子类实现 AbortTask

#### 行为树 UBehaviorTreeComponent 的中断逻辑 - AbortCurrentTask

Behavior Tree Component 每帧 Tick 搜索可执行的 Task，如果能搜索到则调用 AbortCurrentTask() 中断当前 Task。

AbortCurrentTask 内部简介调用 UBT Task 的 AbortTask 来完成中断。

对比了 GameplayTask 和 UBTNode，两者都实现了 IGameplayTaskOwnerInterface，但只有 UBTNode 的子类 BTTaskNode 提供了中断接口，这说明中断行为不是默认提供的，用户要自己编辑逻辑实现中断。

例如，如果用户在自定义的 BT Task 内调用了 GameplayTask，那么需要在 BT Task Abort 时主动取消 GameplayTask，不然中断后 Gameplay Task 会一直执行。

参考：
1. [UE4 ActionGame知识点总结1-FLatentActionInfo以及FLatentActionManager使用-CSDN博客](https://blog.csdn.net/hui211314ddhui/article/details/80710229)
2. [UE4 浅谈FLatentActionManager - 知乎](https://zhuanlan.zhihu.com/p/675932469)
3. [UE代码-游戏AI-行为树执行流程（待更新） - ccsu\_madoka - 博客园](https://www.cnblogs.com/whitelily/p/17100961.html)

### AI 与 Player 打招呼

打招呼行为是视觉上感知的结果，为 AI Controller 挂载 AIPerceptionComponent，为 AI 设置视觉感知监听。

UE 提供了 AI Perception Component 和 AI Perception Stimuli Source Component 这套机制来提供视觉感知效果。
## AI 遇到 Player 伤害行为逃离

AI 伤害感知：UAIScene_Damage::ReportDamageEvent 是一个静态函数，伤害刺激源需要用户主动调用该函数，触发伤害事件广播，配置步骤如下：
1. Perception 组件要配置 AI Damage Scene Config
2. 刺激源调用 ReportDamageEvent，触发 Perception 组件更新
3. Perception 组件触发 OnPerceptionUpdated 事件，参考 [这个教程](https://www.youtube.com/watch?v=g8rz7aZyDMs)，在事件中使用了 GetActorsPerception 函数，遍历所有 UAIScene 来修改行为树参数，值得参考。
	![[AI 不同的行为实现-20240311.png|600]]

AI 逃离行为


参考：
1. [UE4源码-AI感知系统AIPerception（选摘） - 知乎](https://zhuanlan.zhihu.com/p/569297977)
2. [UE4 关于AIPerception（一） - 知乎](https://zhuanlan.zhihu.com/p/463515204)
3. [UE4 关于AIPerception（二） - 知乎](https://zhuanlan.zhihu.com/p/463525577)
4. [Unreal Engine 5 Tutorial - AI Part 3: Perception System - YouTube](https://www.youtube.com/watch?v=bx7taRBjJgM)
