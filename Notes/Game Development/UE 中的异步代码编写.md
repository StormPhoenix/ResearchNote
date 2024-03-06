---
created: 2024-03-05T16:58
updated: 2024-03-05T20:15
---
异步任务的实质是：<mark style="background: #FFB86CA6;">利用回调函数的编程技法，将需要一段时间才能执行完的代码抽象成一个 Task 实例</mark>。举个例子，存在某个 Task 包含 Activate() 和 Deactivate() 函数，Activate() 用于触发我们要执行的任务代码，Deactivate() 函数用于处理当任务结束的后处理，那么需要利用回调编程技法，让任务代码在执行完的时候去执行 Deactivate()。

以上是异步代码的本质。
## UBlueprintAsyncActionBase

Dynamic AI System 插件采用该类实现 Action 的动态配置。

UBlueprintAsyncActionBase 的用法参考下案例：

~~~cpp
UCLASS()
class UAsyncAction : public UBlueprintAsyncActionBase 
{
	GENERATED_BODY()
public:
	/* 蓝图使用此函数创建 Task 节点 */
	UFUNCTION(BlueprintCallable, meta=(BlueprintInternalUseOnly="true"))
	static UAsyncAction* ExecuteAsyncAction();

	/* 真正 Task 执行部分 */
	virtual void Activate() override;

	/* 任务结束后的回调 */
	UPROPERTY(BlueprintAssignable)
	FAsyncActionFinishDelegate OnFinished;
}
~~~

代码用到了 `BlueprintInternalUseOnly` 宏，蓝图能识别这个宏，知道 `UAsyncAction` 将作为一个 Task 运行。目前见到的所有蓝图中调用异步任务的位置，全都添加上了 `BlueprintInternalUseOnly`。

## Gameplay Task

关键类：IGameplayTaskOwnerInterface  和 UGameplayTask，两者配合使用，实现异步任务。

UGameplayTask 就相当于封装好的 Task 类型，IGameplayTaskOwnerInterface 是 Task 的管理者。

凡实现了 IGameplayTaskOwnerInterface 接口的类，皆可以创建 UGameplayTask，然后在合适的时机 (有用户选择) 启动 UGameplayTask，当任务执行完毕后，会回调 IGameplayTaskOwnerInterface 的任务结束接口。

于是一个类型，既能继承 UGameplayTask，又能同时实现 IGameplayTaskOwnerInterface 接口，这个类同时是 Task 又是 Task 的管理者。