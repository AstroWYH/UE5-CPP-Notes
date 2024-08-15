# 方法1：AsyncTask
```cpp
#include "Async/Async.h"

void YourClass::CheckAndExecute(const Data& MyData)
{
    if (bMemberState) // 如果bMemberState满足，直接执行
    {
        Fun1(MyData);
        return;
    }

    AsyncTask(ENamedThreads::AnyBackgroundThreadNormalTask, [this, MyData]()
    {
        while (!bMemberState) // 异步线程空跑，等待bMemberState状态满足
        {
            FPlatformProcess::Sleep(0.1f); // 避免线程性能浪费
        }

        AsyncTask(ENamedThreads::GameThread, [this, MyData]()
        {
            Fun1(); // 这里是等到成员变量状态后，回到主线程处理。如果不需要，也可以直接执行Fun1()
        });
    });
}

void YourClass::Fun1()
{
    // 你的函数实现
}
```

# 方法2：CreateAndDispatchWhenReady
```cpp
void CheckAndExecute(const FZCProtocol_notify_raid_spawners& ProtocolObject)
{
    auto AsyncTask = [ProtocolObject]()
    {
        while (!bMemberState)
        {
            FPlatformProcess::Sleep(0.01f);
        }

        Func1(ProtocolObjectCopy);
    };

    // 用CreateAndDispatchWhenReady开启异步线程
    FFunctionGraphTask::CreateAndDispatchWhenReady(AsyncTask, TStatId(), nullptr, ENamedThreads::AnyBackgroundThreadNormalTask);
}
```
