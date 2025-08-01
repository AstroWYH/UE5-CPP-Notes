<img width="1759" height="1040" alt="image" src="https://github.com/user-attachments/assets/80028f7d-ea6b-4596-aaf7-5f4d5584d085" />

```cpp
#include "EveThreadGameModeBase.h"

uint32 GetThreadId()
{
	return FPlatformTLS::GetCurrentThreadId();
}

void AEveThreadGameModeBase::InitGameState()
{
	Super::InitGameState();

	UE_LOG(LogTemp, Display, TEXT("=== Multithreading Demo Start [MainThreadId: %u] ==="), GetThreadId());

	// TaskGraph 测试
	{
		UE_LOG(LogTemp, Display, TEXT("[TaskGraph] Start on Thread %u"), GetThreadId());

		FGraphEventRef Task = FFunctionGraphTask::CreateAndDispatchWhenReady([]()
		{
			UE_LOG(LogTemp, Display, TEXT("[TaskGraph] Task running on Thread %u"), GetThreadId());
			FPlatformProcess::Sleep(0.5f);
			UE_LOG(LogTemp, Display, TEXT("[TaskGraph] Task complete on Thread %u"), GetThreadId());
		});

		FTaskGraphInterface::Get().WaitUntilTaskCompletes(Task);
		UE_LOG(LogTemp, Display, TEXT("[TaskGraph] Finished on Thread %u"), GetThreadId());
	}

	// Async 测试
	{
		UE_LOG(LogTemp, Display, TEXT("[Async] Start on Thread %u"), GetThreadId());

		TFuture<void> Future = Async(EAsyncExecution::Thread, []()
		{
			UE_LOG(LogTemp, Display, TEXT("[Async] Task running on Thread %u"), GetThreadId());
			FPlatformProcess::Sleep(0.5f);
			UE_LOG(LogTemp, Display, TEXT("[Async] Task complete on Thread %u"), GetThreadId());
		});

		Future.Wait();
		UE_LOG(LogTemp, Display, TEXT("[Async] Finished on Thread %u"), GetThreadId());
	}

	// ParallelFor 测试
	{
		UE_LOG(LogTemp, Display, TEXT("[ParallelFor] Start on Thread %u"), GetThreadId());

		ParallelFor(4, [](int32 Index)
		{
			UE_LOG(LogTemp, Display, TEXT("[ParallelFor] Index %d running on Thread %u"), Index, GetThreadId());
			FPlatformProcess::Sleep(0.2f);
			UE_LOG(LogTemp, Display, TEXT("[ParallelFor] Index %d complete on Thread %u"), Index, GetThreadId());
		});

		UE_LOG(LogTemp, Display, TEXT("[ParallelFor] Finished on Thread %u"), GetThreadId());
	}

	// Runnable 测试
	{
		UE_LOG(LogTemp, Display, TEXT("[Runnable] Start on Thread %u"), GetThreadId());

		class FTestRunnable : public FRunnable
		{
		public:
			virtual uint32 Run() override
			{
				UE_LOG(LogTemp, Display, TEXT("[Runnable] Task running on Thread %u"), GetThreadId());
				FPlatformProcess::Sleep(0.5f);
				UE_LOG(LogTemp, Display, TEXT("[Runnable] Task complete on Thread %u"), GetThreadId());
				return 0;
			}
		};

		FTestRunnable* Runnable = new FTestRunnable();
		FRunnableThread* Thread = FRunnableThread::Create(Runnable, TEXT("TestRunnableThread"));

		if (Thread)
		{
			Thread->WaitForCompletion();
			delete Thread;
		}
		delete Runnable;

		UE_LOG(LogTemp, Display, TEXT("[Runnable] Finished on Thread %u"), GetThreadId());
	}

	UE_LOG(LogTemp, Display, TEXT("=== Multithreading Demo End [MainThreadId: %u] ==="), GetThreadId());
}

```

