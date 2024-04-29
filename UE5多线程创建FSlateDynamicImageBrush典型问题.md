问题流程：

```cpp
OnMapXXXChanged（可能多次触发，第1次进入）
 -->MapResourceManager::Get()->CurrentBackgroundBrush.Empty();
	MapResourceManager::Get()->RequestRouteImage(MapIdStr, LevelStr, LayerId);

// CurrentBackgroundBrush.Empty()
FSlateDynamicImageBrush::~FSlateDynamicImageBrush( )
{
    // 析构原有Image
 -->ReleaseResourceInternal();
}

void FDynamicResourceMap::RemoveDynamicTextureResource(FName ResourceName)
{
    // NativeTextureMap中删掉了Key这一条
 -->NativeTextureMap.Remove(ResourceName);
}

OnMapXXXChanged（可能多次触发，第1次进入）
    MapResourceManager::Get()->CurrentBackgroundBrush.Empty();
 -->MapResourceManager::Get()->RequestRouteImage(MapIdStr, LevelStr, LayerId);

void MapResourceManager::TryAsyncLoadImage(FImage* Img)
{
	FTask* RequestTask = new FTask(Img);
    // 注意，这里很容易出问题，因为后续多次触发Request时，Img->GetId()是一样的
    // AllLoadRouteTasks_GameThread里如果没有Remove，则很可能Add覆盖
    // 则会出现Request多次，实际Create一次（或大于一次，但少于多次）的情况。
 -->AllLoadRouteTasks_GameThread.Add(Img->GetId(), RequestTask);
	RouteRunnable->AddRequestTask_GameThread(Img->GetId(), RequestTask);
	RouteWaitEvent->Trigger();
}

void MapResourceManager::Tick(float fDelta)
{
    // 可能隔了2帧，正式开始Create
 -->TSharedPtr<FSlateDynamicImageBrush> Brush = FSlateDynamicImageBrush::CreateWithImageData2(xxx, ...);
}

bool FSlateRHIRenderer::GenerateDynamicImageResource2(FName ResourceName, uint32 Width, uint32 Height, const TArray< uint8 >& Bytes, bool bSRGB, EPixelFormat Fmt)
{
 -->TSharedPtr<FSlateDynamicTextureResource> TextureResource = 
        ResourceManager->GetDynamicTextureResourceByName(ResourceName);
    // TextureResource此时为nullptr，正常现象，因为之前毕竟NativeTextureMap.Remove了
	if (!TextureResource.IsValid())
	{
        // 可以正常走进这里，重新Create
 -->	TextureResource = ResourceManager->MakeDynamicTextureResource2(
            ResourceName, Width, Height, Bytes, bSRGB, Fmt);
	}
	return TextureResource.IsValid();
}

// 又Add到NativeTextureMap里
-->DynamicResourceMap.AddDynamicTextureResource( ResourceName, TextureResource.ToSharedRef() );

// 正常流程：打开M界面，完整进行了一次Remove和Create，重新回到M界面。
// 走第1次OnMapXXXChanged，然后第1次Empty()，然后第1次Create。

// 异常流程1：在MapResourceManager::Tick->CreateWithImageData2之前，可能会第2次OnMapXXXChanged，
// 然后先执行第2次CurrentBackgroundBrush.Empty()，因为第1次的Create还没执行到，所以第2次Empty()了个寂寞，
// 然后又执行第2次RequestRouteImage，在TryAsyncLoadImage时，AllLoadRouteTasks_GameThread.Add会后者覆盖前者，
// 因为TMap不是MultiMap，只会把原来Task替换掉，后面取Task去第1次Create，第2次Create就被吃了，有可能执行Task1，
// 有可能执行Task2，但Task1=Task2。然后看起来，是正常的执行了一次Create流程。
// 异常流程1就是现有大部分时间的流程，一般不会有问题。

// 异常流程2：在MapResourceManager::Tick->CreateWithImageData2之前，可能会第2次OnMapXXXChanged
// 然后先执行第2次CurrentBackgroundBrush.Empty()，因为第1次Create还没执行到，所以Empty了个寂寞（这都和异常流程1一样）
// 然后又执行第2次RequestRouteImage，在TryAsyncLoadImage时，AllLoadRouteTasks_GameThread.Add会直接Add成功
// 这是因为此时第1次的Create流程已经完整结束，AllLoadRouteTasks_GameThread.Remove已经执行完了
// 然后去成功执行了第2次的Create，此时问题出现，因为第2次Empty执行了寂寞，然后第1次Create执行成功，所以第2次Create
// 会判断当前NativeTextureMap是存在Key的，其实这虽然流程错了，但也还没导致问题。然后第3次OnMapXXXChanged，执行了
// 第3次CurrentBackgroundBrush.Empty()，会先在游戏线程从NativeTextureMap.Remove(Key)，然后排入渲染线程队列，
// 后续会把第2次Create的结果析构了（导致白图）。回到刚才第3次执行RequestRouteImage，在TryAsyncLoadImage时，第2次
// Create流程还没完全结束，AllLoadRouteTasks_GameThread还没清理，Add只是覆盖，所以第3次Create不会执行。
// 所以最终导致了白图，异常流程2就是偶现Bug。
```

