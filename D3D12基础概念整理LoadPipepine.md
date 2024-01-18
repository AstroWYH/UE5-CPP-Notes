```cpp
//*********************************************************
//
// 版权所有 (c) Microsoft。保留所有权利。
// 此代码根据 MIT 许可证（MIT）获得许可。
// 此代码按 "原样" 提供，无任何形式的担保，
// 包括任何对特定目的适用性的暗示担保、商业性或非侵权性的。
//
//*********************************************************

// 定义输入结构体 PSInput，包含顶点位置和颜色
struct PSInput
{
    float4 position : SV_POSITION; // 顶点位置
    float4 color : COLOR;           // 顶点颜色
};

// 顶点着色器函数，将顶点位置和颜色传递到像素着色器
PSInput VSMain(float4 position : POSITION, float4 color : COLOR)
{
    PSInput result;

    result.position = position;
    result.color = color;

    return result;
}

// 像素着色器函数，返回最终像素颜色
float4 PSMain(PSInput input) : SV_TARGET
{
    return input.color;
}

```

1. `struct PSInput`：定义了一个结构体，用于存储顶点着色器传递到像素着色器的数据。包含了顶点位置和颜色信息。
2. `VSMain` 函数：顶点着色器的主函数，接收顶点位置和颜色作为输入，填充 `PSInput` 结构体，并返回该结构体。在这里，仅仅是将输入的位置和颜色传递到结构体中。
3. `PSMain` 函数：像素着色器的主函数，接收 `PSInput` 结构体作为输入，返回最终的像素颜色。在这里，简单地返回结构体中的颜色信息。

注：`SV_POSITION` 和 `SV_TARGET` 是语义标记，用于指定顶点位置和像素着色器的输出目标。



## command queue

在图形编程中，特别是在使用底层图形API（如DirectX 12）时，"command queue"（命令队列）是一种用于提交和执行图形渲染命令的机制。命令队列是GPU命令的发出和执行的通道，用于处理渲染命令的提交和异步执行。

在DirectX 12中，存在两种主要类型的命令队列：图形命令队列（Graphics Command Queue）和计算命令队列（Compute Command Queue）。这两种类型的队列允许分别处理图形渲染和计算任务，提供了更灵活的并行性。

以下是关于命令队列的一些要点：

1. **图形命令队列（Graphics Command Queue）：** 主要用于处理与图形渲染相关的命令，例如绘制几何体、设置渲染目标等。
2. **计算命令队列（Compute Command Queue）：** 用于执行与通用计算相关的任务，如计算着色器、图像处理等。
3. **命令列表（Command List）：** 为了提交到命令队列，需要首先创建一个命令列表，将所有需要执行的命令添加到该列表中。
4. **命令分配器（Command Allocator）：** 命令分配器用于为命令列表分配内存。每个命令列表都需要有一个关联的命令分配器。
5. **命令队列的执行：** 通过将命令列表提交到命令队列中，GPU将异步执行这些命令。可以通过多个命令列表和多个命令队列来实现并行处理，提高性能。

使用命令队列的主要目的是实现异步渲染和计算，从而充分发挥GPU的并行性能。通过在主线程上提交命令列表到命令队列，并在后台异步执行，应用程序能够在处理渲染任务的同时进行其他工作，提高整体性能和响应性。



## Swap Chain

```cpp
    // Describe and create the swap chain.
    DXGI_SWAP_CHAIN_DESC1 swapChainDesc = {};
    swapChainDesc.BufferCount = FrameCount;
    swapChainDesc.Width = m_width;
    swapChainDesc.Height = m_height;
    swapChainDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
    swapChainDesc.BufferUsage = DXGI_USAGE_RENDER_TARGET_OUTPUT;
    swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_FLIP_DISCARD;
    swapChainDesc.SampleDesc.Count = 1;    

	ComPtr<IDXGISwapChain1> swapChain;
    ThrowIfFailed(factory->CreateSwapChainForHwnd(
        m_commandQueue.Get(),        // Swap chain needs the queue so that it can force a flush on it.
        Win32Application::GetHwnd(),
        &swapChainDesc,
        nullptr,
        nullptr,
        &swapChain
        ));
```

在DirectX 12（DX12）中，Swap Chain（交换链）是用于处理前后缓冲切换的重要概念。交换链允许应用程序在后台渲染新的图像时，同时将已经渲染好的图像呈现到屏幕上，实现平滑的动画和用户交互。

交换链的作用包括：

1. **双缓冲和三缓冲：** 交换链允许使用双缓冲或三缓冲机制。在双缓冲中，有两个缓冲区，一个用于渲染，一个用于显示。在三缓冲中，有三个缓冲区，其中一个用于显示，一个用于渲染，另一个在后台用于异步渲染，可以提高性能和平滑度。
2. **解决撕裂问题：** 交换链还有助于解决屏幕撕裂（screen tearing）问题。屏幕撕裂是由于在渲染新图像的同时，显示器正在读取上一帧的图像，导致图像不一致。通过使用交换链，可以将新的图像呈现到不可见的缓冲区，然后在垂直同步（V-Sync）时切换到可见的缓冲区，从而避免了撕裂问题。

在DX12中，创建和配置Swap Chain通常需要以下步骤：

1. **创建交换链描述对象（`DXGI_SWAP_CHAIN_DESC`）：** 包含有关交换链的各种设置，如分辨率、格式、缓冲区数量等。
2. **创建交换链：** 使用`IDXGIFactory4`接口的`CreateSwapChainForHwnd`方法创建交换链。
3. **获取交换链中的缓冲区：** 通过`GetBuffer`方法获取交换链中的后台缓冲区，用于渲染。
4. **设置交换链的呈现目标：** 使用`IDXGISwapChain3`接口的`Present`方法将渲染好的图像呈现到屏幕上。

在DX12中，与DX11相比，交换链的创建和管理更为灵活，因为它允许应用程序更精细地控制渲染和显示的过程。



## Descriptor Heaps

```cpp
    // Create descriptor heaps.
    {
        // Describe and create a render target view (RTV) descriptor heap.
        D3D12_DESCRIPTOR_HEAP_DESC rtvHeapDesc = {};
        rtvHeapDesc.NumDescriptors = FrameCount;
        rtvHeapDesc.Type = D3D12_DESCRIPTOR_HEAP_TYPE_RTV;
        rtvHeapDesc.Flags = D3D12_DESCRIPTOR_HEAP_FLAG_NONE;
        ThrowIfFailed(m_device->CreateDescriptorHeap(&rtvHeapDesc, IID_PPV_ARGS(&m_rtvHeap)));

        m_rtvDescriptorSize = m_device->GetDescriptorHandleIncrementSize(D3D12_DESCRIPTOR_HEAP_TYPE_RTV);
    }
```

在图形编程中，特别是在使用DirectX 12（DX12）等底层图形API时，Descriptor Heaps（描述符堆）是一种用于管理着色器资源描述符的数据结构。描述符指的是用于描述着色器需要使用的资源（如纹理、缓冲区、采样器等）的信息。Descriptor Heaps 提供了一种有序和高效的方式来管理这些描述符。

在DX12中，着色器资源描述符包括以下几种类型：

1. **CBV（Constant Buffer View）：** 用于描述常量缓冲区的视图。
2. **SRV（Shader Resource View）：** 用于描述着色器读取的资源视图，如纹理和缓冲区。
3. **UAV（Unordered Access View）：** 用于描述可以被着色器读写的资源视图，通常用于图像处理等需要写入的场景。
4. **Sampler：** 用于描述采样器状态，例如纹理过滤和寻址模式。

Descriptor Heaps 有助于组织和管理这些描述符，使得在渲染管线中轻松地绑定和切换这些资源。

在DX12中，可以使用 `ID3D12DescriptorHeap` 接口来创建和管理 Descriptor Heaps。Descriptor Heaps 是由一个或多个描述符堆（Descriptor Heap）组成的，每个描述符堆都专门用于一种类型的描述符（CBV、SRV、UAV 或 Sampler）。

使用 Descriptor Heaps 的一般步骤包括：

1. **创建 Descriptor Heap：** 通过调用 `ID3D12Device::CreateDescriptorHeap` 来创建 Descriptor Heap。
2. **描述符分配：** 使用 `ID3D12DescriptorHeap` 接口的 `GetCPUDescriptorHandleForHeapStart` 或 `GetGPUDescriptorHandleForHeapStart` 方法获取描述符堆的起始CPU或GPU句柄，并根据需要进行描述符分配。
3. **在渲染过程中绑定描述符：** 在渲染管线的不同阶段，将描述符绑定到着色器。

通过使用 Descriptor Heaps，开发人员可以更有效地管理着色器资源，避免了手动跟踪和维护描述符的复杂性。这在底层图形编程中是一项非常有用的优化和管理技术。



## frame resources

```cpp
    // Create frame resources.
    {
        CD3DX12_CPU_DESCRIPTOR_HANDLE rtvHandle(m_rtvHeap->GetCPUDescriptorHandleForHeapStart());

        // Create a RTV for each frame.
        for (UINT n = 0; n < FrameCount; n++)
        {
            ThrowIfFailed(m_swapChain->GetBuffer(n, IID_PPV_ARGS(&m_renderTargets[n])));
            m_device->CreateRenderTargetView(m_renderTargets[n].Get(), nullptr, rtvHandle);
            rtvHandle.Offset(1, m_rtvDescriptorSize);
        }
    }
```

在图形编程中，特别是在使用底层图形API（如DirectX 12）进行渲染时，"frame resources"（帧资源）是一种用于管理每一帧所需的资源的概念。这是一种有效的资源管理策略，通常用于在多帧渲染中有效地处理资源的创建、使用和释放。

每一帧都可能需要不同的资源，例如常量缓冲区（Constant Buffers）、纹理、顶点缓冲区等。为了避免资源冲突和浪费，以及提高性能，可以使用帧资源来管理这些动态变化的资源。

一般来说，帧资源的管理流程包括以下步骤：

1. **创建帧资源结构：** 定义一个结构或类，其中包含当前帧需要的所有资源，如常量缓冲区、纹理等。
2. **创建多个帧资源：** 在应用程序初始化时，创建多个帧资源的实例，通常与帧缓冲数目相等（例如，三个帧缓冲对应三个帧资源）。
3. **在每一帧开始时更新帧资源：** 在每一帧开始渲染之前，更新当前帧资源中的数据，以确保正确的资源被使用。这可能涉及到设置常量缓冲区的新数据、更换纹理等操作。
4. **在渲染管线中使用当前帧资源：** 在渲染过程中，使用当前帧资源的数据，例如在着色器中绑定常量缓冲区，将当前帧的纹理传递给渲染过程等。
5. **在帧结束时释放资源：** 在帧结束时，确保释放当前帧资源中的不再需要的资源，以便在下一帧中重新分配。

这种方法的好处是，它允许在每一帧中灵活地管理和更新所需的资源，同时防止资源的重复创建和销毁，提高了性能。这对于动态场景、动画和实时交互等情况非常有用。

## Debug Layer

在DirectX 12和其他一些图形API中，Debug Layer（调试层）是一个用于帮助开发人员识别和调试图形应用程序中潜在问题的工具。Debug Layer 提供了一系列检查和报告机制，能够捕获并报告应用程序中的潜在错误、性能问题和不良实践。

在你提供的代码中，通过使用 `_DEBUG` 宏，开发者可以在调试模式下启用Debug Layer。以下是对代码的解释：

```cpp
#if defined(_DEBUG)
    // 在调试模式下启用调试层
    // 注意：在设备创建后启用调试层会使激活的设备失效。
    {
        ComPtr<ID3D12Debug> debugController;
        if (SUCCEEDED(D3D12GetDebugInterface(IID_PPV_ARGS(&debugController))))
        {
            debugController->EnableDebugLayer();

            // 启用额外的调试层。
            dxgiFactoryFlags |= DXGI_CREATE_FACTORY_DEBUG;
        }
    }
#endif

```

1. 在调试模式下，尝试获取 `ID3D12Debug` 接口，这是用于启用Debug Layer的接口。
2. 如果成功获取到 `ID3D12Debug` 接口，通过调用 `EnableDebugLayer` 方法启用调试层。
3. 启用额外的调试层标志 `DXGI_CREATE_FACTORY_DEBUG`。这个标志用于在后续创建 DXGI 工厂时启用调试。

启用Debug Layer后，将会在运行时捕获并输出调试信息，包括但不限于：

- 错误消息：有关渲染调用中发生的错误的详细信息。
- 警告消息：与不良实践或性能问题相关的警告。
- 实时性能信息：关于GPU使用和渲染性能的信息。

在开发和调试阶段，启用调试层是非常有用的，因为它有助于及早发现和解决潜在的问题。但在最终发布的版本中，通常会禁用调试层以提高性能。

## LoadPipepline全代码

```cpp
// Load the rendering pipeline dependencies.
void D3D12HelloTriangle::LoadPipeline()
{
    UINT dxgiFactoryFlags = 0;

#if defined(_DEBUG)
    // Enable the debug layer (requires the Graphics Tools "optional feature").
    // NOTE: Enabling the debug layer after device creation will invalidate the active device.
    {
        ComPtr<ID3D12Debug> debugController;
        if (SUCCEEDED(D3D12GetDebugInterface(IID_PPV_ARGS(&debugController))))
        {
            debugController->EnableDebugLayer();

            // Enable additional debug layers.
            dxgiFactoryFlags |= DXGI_CREATE_FACTORY_DEBUG;
        }
    }
#endif

    ComPtr<IDXGIFactory4> factory;
    ThrowIfFailed(CreateDXGIFactory2(dxgiFactoryFlags, IID_PPV_ARGS(&factory)));

    if (m_useWarpDevice)
    {
        ComPtr<IDXGIAdapter> warpAdapter;
        ThrowIfFailed(factory->EnumWarpAdapter(IID_PPV_ARGS(&warpAdapter)));

        ThrowIfFailed(D3D12CreateDevice(
            warpAdapter.Get(),
            D3D_FEATURE_LEVEL_11_0,
            IID_PPV_ARGS(&m_device)
            ));
    }
    else
    {
        ComPtr<IDXGIAdapter1> hardwareAdapter;
        GetHardwareAdapter(factory.Get(), &hardwareAdapter);

        ThrowIfFailed(D3D12CreateDevice(
            hardwareAdapter.Get(),
            D3D_FEATURE_LEVEL_11_0,
            IID_PPV_ARGS(&m_device)
            ));
    }

    // Describe and create the command queue.
    D3D12_COMMAND_QUEUE_DESC queueDesc = {};
    queueDesc.Flags = D3D12_COMMAND_QUEUE_FLAG_NONE;
    queueDesc.Type = D3D12_COMMAND_LIST_TYPE_DIRECT;

    ThrowIfFailed(m_device->CreateCommandQueue(&queueDesc, IID_PPV_ARGS(&m_commandQueue)));

    // Describe and create the swap chain.
    DXGI_SWAP_CHAIN_DESC1 swapChainDesc = {};
    swapChainDesc.BufferCount = FrameCount;
    swapChainDesc.Width = m_width;
    swapChainDesc.Height = m_height;
    swapChainDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
    swapChainDesc.BufferUsage = DXGI_USAGE_RENDER_TARGET_OUTPUT;
    swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_FLIP_DISCARD;
    swapChainDesc.SampleDesc.Count = 1;

    ComPtr<IDXGISwapChain1> swapChain;
    ThrowIfFailed(factory->CreateSwapChainForHwnd(
        m_commandQueue.Get(),        // Swap chain needs the queue so that it can force a flush on it.
        Win32Application::GetHwnd(),
        &swapChainDesc,
        nullptr,
        nullptr,
        &swapChain
        ));

    // This sample does not support fullscreen transitions.
    ThrowIfFailed(factory->MakeWindowAssociation(Win32Application::GetHwnd(), DXGI_MWA_NO_ALT_ENTER));

    ThrowIfFailed(swapChain.As(&m_swapChain));
    m_frameIndex = m_swapChain->GetCurrentBackBufferIndex();

    // Create descriptor heaps.
    {
        // Describe and create a render target view (RTV) descriptor heap.
        D3D12_DESCRIPTOR_HEAP_DESC rtvHeapDesc = {};
        rtvHeapDesc.NumDescriptors = FrameCount;
        rtvHeapDesc.Type = D3D12_DESCRIPTOR_HEAP_TYPE_RTV;
        rtvHeapDesc.Flags = D3D12_DESCRIPTOR_HEAP_FLAG_NONE;
        ThrowIfFailed(m_device->CreateDescriptorHeap(&rtvHeapDesc, IID_PPV_ARGS(&m_rtvHeap)));

        m_rtvDescriptorSize = m_device->GetDescriptorHandleIncrementSize(D3D12_DESCRIPTOR_HEAP_TYPE_RTV);
    }

    // Create frame resources.
    {
        CD3DX12_CPU_DESCRIPTOR_HANDLE rtvHandle(m_rtvHeap->GetCPUDescriptorHandleForHeapStart());

        // Create a RTV for each frame.
        for (UINT n = 0; n < FrameCount; n++)
        {
            ThrowIfFailed(m_swapChain->GetBuffer(n, IID_PPV_ARGS(&m_renderTargets[n])));
            m_device->CreateRenderTargetView(m_renderTargets[n].Get(), nullptr, rtvHandle);
            rtvHandle.Offset(1, m_rtvDescriptorSize);
        }
    }

    ThrowIfFailed(m_device->CreateCommandAllocator(D3D12_COMMAND_LIST_TYPE_DIRECT, IID_PPV_ARGS(&m_commandAllocator)));
}
```
