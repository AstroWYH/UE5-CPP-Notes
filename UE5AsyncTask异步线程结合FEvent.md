```cpp
void UXXSubsystem::Func(const ProObj& ProtocolObject)
{
	// bFlag会在主线程其他地方置true，可能比Func早，可能更晚
	// bFlag需要是std::atomic<bool>
	// 一旦bFlag被置true，则使用FEvent->Nofity()
	if (bFlag)
	{
		Process(ProtocolObject);
		return;
	}

	// 如果bFlag还未置true，则开启异步线程等待
	AsyncTask(ENamedThreads::AnyBackgroundThreadNormalTask, [this, ProtocolObject]()
		{
			// 设置超时，避免一直等待
			const float TimeoutSeconds = 30.0f;
			const double StartTime = FPlatformTime::Seconds();
			// 取消while循环等待，使用FEvent->Wait()，并设置超时
			while (!bFlag)
			{
				if (FPlatformTime::Seconds() - StartTime > TimeoutSeconds)
				{
					UE_LOG(XXX, Error, TEXT("Time Out Error, bFlag:%d after %.2f seconds."), bFlag.load(), FPlatformTime::Seconds() - StartTime);
					return;
				}

				FPlatformProcess::Sleep(0.1f);
			}

			// 最后回到主线程，处理业务逻辑，避免异步线程和主线程发生数据竞争
			AsyncTask(ENamedThreads::GameThread, [this, ProtocolObject]()
				{
					Process(ProtocolObject);
				});
		});
}

```
FEvent获取需要从GetPool，返回同理ReturnPool
FEvent需要注意，如果已经Trigger过了，那始终无法再Wait阻塞成功，除非手动Reset
