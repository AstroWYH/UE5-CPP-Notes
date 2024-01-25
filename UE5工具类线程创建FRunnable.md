这里写得不够完整，大致思路如下。

1. 写一个自定义类继承FRunnable类UMyRunnable。
2. 在初始化时，MakeShared该类， XXXRunnable = MakeShared<UMyRunnable>(this)。并创建通过FRunnableThread::Create(XXXRunnable， TEXT("XXXName"))（可选）。
3. 可创建一个FEvent waitevent = FGenericPlatformProcess::GetSynchEventFromPool();，这个类似cv条件变量，可用于wait和notify。
4. 注意，UMyRunnable在创建后，Run函数在任务线程执行，其他自定义的函数，在主线程执行。

```c
#include "MyRunnable.h"

UMyRunnable::UMyRunnable()
{
    // 创建 FRunnableThread，并启动线程
    MyThread = FRunnableThread::Create(this, TEXT("MyThread"), 0, TPri_BelowNormal);
}

UMyRunnable::~UMyRunnable()
{
    // 在对象销毁时，等待线程结束
    if (MyThread)
    {
        MyThread->WaitForCompletion();
        delete MyThread;
        MyThread = nullptr;
    }
}

bool UMyRunnable::Init()
{
    // 初始化，如果返回 false，线程将立即退出
    return true;
}

uint32 UMyRunnable::Run()
{
    // 在这里执行后台线程的逻辑
    while (bIsRunning)
    {
        // Do something...

        FPlatformProcess::Sleep(0.1f);  // 避免循环过快
    }

    return 0;
}

void UMyRunnable::Stop()
{
    // 设置标志，使线程退出循环
    bIsRunning = false;
}

```

