## Root Signature（根签名）

```c
// Create an empty root signature.
{
    CD3DX12_ROOT_SIGNATURE_DESC rootSignatureDesc;
    rootSignatureDesc.Init(0, nullptr, 0, nullptr, D3D12_ROOT_SIGNATURE_FLAG_ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT);

    ComPtr<ID3DBlob> signature;
    ComPtr<ID3DBlob> error;
    ThrowIfFailed(D3D12SerializeRootSignature(&rootSignatureDesc, D3D_ROOT_SIGNATURE_VERSION_1, &signature, &error));
    ThrowIfFailed(m_device->CreateRootSignature(0, signature->GetBufferPointer(), signature->GetBufferSize(), IID_PPV_ARGS(&m_rootSignature)));
}
```

在图形编程中，特别是在使用DirectX 12等底层图形API时，Root Signature（根签名）是一种描述GPU着色器需要访问的资源的机制。根签名定义了着色器要使用的资源，如常量缓冲区、纹理、采样器等。在DirectX 12中，根签名是GPU着色器和管线之间的接口，它告诉GPU在渲染过程中如何访问和绑定资源。

以下是一些根签名的关键概念：

1. **描述着色器资源：** 根签名定义了GPU着色器需要的所有资源，包括常量缓冲区、纹理、采样器等。这些描述符告诉GPU在渲染时如何访问这些资源。
2. **绑定根签名：** 在渲染流水线中，根签名需要与着色器一起绑定，以便GPU能够正确地访问资源。在DirectX 12中，可以使用 `ID3D12GraphicsCommandList` 接口的 `SetGraphicsRootSignature` 或 `SetComputeRootSignature` 方法进行绑定。
3. **版本和硬件支持：** 根签名的版本和硬件支持可能因不同的DirectX版本和GPU而有所不同。DirectX 12引入了新的根签名概念，相较于DirectX 11的常量缓冲区和纹理插槽，提供了更灵活和精细的资源管理。
4. **管理着色器常量：** 根签名通常还包括用于传递着色器常量数据的常量缓冲区描述符。这些常量描述符告诉GPU在渲染时如何访问常量数据。

下面是一个简单的根签名的示例：

```c
// 着色器常量结构体
struct ShaderConstants
{
    float4 color;
    float   intensity;
};

// 根签名定义
RootSignature MyRootSignature
{
    // 常量缓冲区描述符
    ConstantBuffer<ShaderConstants> MyConstantBuffer;
    
    // 纹理和采样器描述符
    Texture2D MyTexture;
    SamplerState MySampler;
};

```

这个示例中，`MyRootSignature` 定义了一个包含常量缓冲区、纹理和采样器描述符的根签名。在实际应用中，根签名的定义会根据实际需要而有所不同。

## pipeline state

```c
    // Create the pipeline state, which includes compiling and loading shaders.
    {
        ComPtr<ID3DBlob> vertexShader;
        ComPtr<ID3DBlob> pixelShader;

#if defined(_DEBUG)
        // Enable better shader debugging with the graphics debugging tools.
        UINT compileFlags = D3DCOMPILE_DEBUG | D3DCOMPILE_SKIP_OPTIMIZATION;
#else
        UINT compileFlags = 0;
#endif

        ThrowIfFailed(D3DCompileFromFile(GetAssetFullPath(L"shaders.hlsl").c_str(), nullptr, nullptr, "VSMain", "vs_5_0", compileFlags, 0, &vertexShader, nullptr));
        ThrowIfFailed(D3DCompileFromFile(GetAssetFullPath(L"shaders.hlsl").c_str(), nullptr, nullptr, "PSMain", "ps_5_0", compileFlags, 0, &pixelShader, nullptr));

        // Define the vertex input layout.
        D3D12_INPUT_ELEMENT_DESC inputElementDescs[] =
        {
            { "POSITION", 0, DXGI_FORMAT_R32G32B32_FLOAT, 0, 0, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 },
            { "COLOR", 0, DXGI_FORMAT_R32G32B32A32_FLOAT, 0, 12, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 }
        };

        // Describe and create the graphics pipeline state object (PSO).
        D3D12_GRAPHICS_PIPELINE_STATE_DESC psoDesc = {};
        psoDesc.InputLayout = { inputElementDescs, _countof(inputElementDescs) };
        psoDesc.pRootSignature = m_rootSignature.Get();
        psoDesc.VS = CD3DX12_SHADER_BYTECODE(vertexShader.Get());
        psoDesc.PS = CD3DX12_SHADER_BYTECODE(pixelShader.Get());
        psoDesc.RasterizerState = CD3DX12_RASTERIZER_DESC(D3D12_DEFAULT);
        psoDesc.BlendState = CD3DX12_BLEND_DESC(D3D12_DEFAULT);
        psoDesc.DepthStencilState.DepthEnable = FALSE;
        psoDesc.DepthStencilState.StencilEnable = FALSE;
        psoDesc.SampleMask = UINT_MAX;
        psoDesc.PrimitiveTopologyType = D3D12_PRIMITIVE_TOPOLOGY_TYPE_TRIANGLE;
        psoDesc.NumRenderTargets = 1;
        psoDesc.RTVFormats[0] = DXGI_FORMAT_R8G8B8A8_UNORM;
        psoDesc.SampleDesc.Count = 1;
        ThrowIfFailed(m_device->CreateGraphicsPipelineState(&psoDesc, IID_PPV_ARGS(&m_pipelineState)));
    }
```

在图形编程中，Pipeline State（管线状态）是一组配置，定义了图形渲染管线中的各个阶段的状态和设置。在你提供的代码中，`m_pipelineState` 可以理解为一个用于描述图形管线状态的对象，通常被称为Graphics Pipeline State Object（PSO，图形管线状态对象）。

管线状态包括了图形渲染过程中的多个关键方面，比如顶点着色器、像素着色器、输入布局、光栅化器状态、混合状态、深度模板状态等等。通过配置这些状态，可以影响渲染的结果和性能。

以下是代码中创建管线状态对象（PSO）的主要步骤和概念：

1. **Shader Compilation（着色器编译）：** 使用 `D3DCompileFromFile` 函数编译顶点着色器和像素着色器。这些着色器是由 HLSL（High-Level Shader Language） 编写的，用于定义渲染阶段的计算。
2. **Input Layout（输入布局）：** 定义了顶点数据的格式，即顶点着色器所期望的输入。在这里，使用 `D3D12_INPUT_ELEMENT_DESC` 数组描述了每个顶点的属性，如位置和颜色。
3. **Root Signature（根签名）：** 指定了着色器使用的资源（如常量缓冲区、纹理等）和根常量。在这里，`pRootSignature` 字段指向之前创建的根签名对象。
4. **Rasterizer State（光栅化器状态）：** 定义了光栅化阶段的配置，包括剪裁、多边形填充模式等。在这里，使用了默认的光栅化器状态。
5. **Blend State（混合状态）：** 定义了颜色混合和透明度混合的配置。在这里，使用了默认的混合状态。
6. **Depth-Stencil State（深度模板状态）：** 定义了深度测试和模板测试的配置。在这里，深度测试和模板测试都被禁用。
7. **其他设置：** 包括了图元拓扑类型、渲染目标格式、采样设置等。
8. **创建 Graphics Pipeline State Object（PSO）：** 使用 `CreateGraphicsPipelineState` 方法创建图形管线状态对象。这个对象包含了以上所有配置信息，用于后续的图形渲染。

通过配置这些管线状态，你可以控制图形渲染的方方面面，从而实现不同的渲染效果。

## command list

```c
    // Create the command list.
    ThrowIfFailed(m_device->CreateCommandList(0, D3D12_COMMAND_LIST_TYPE_DIRECT, m_commandAllocator.Get(), m_pipelineState.Get(), IID_PPV_ARGS(&m_commandList)));

    // Command lists are created in the recording state, but there is nothing
    // to record yet. The main loop expects it to be closed, so close it now.
    ThrowIfFailed(m_commandList->Close());
```

在图形编程中，Command List（命令列表）是一个包含要由GPU执行的一系列图形命令的对象。它包含了一组渲染或计算命令，用于定义一帧中要绘制的图形内容或执行的计算任务。在DirectX 12中，Command List 是与命令队列（Command Queue）结合使用的重要概念。

**Command List 的主要作用包括：**

1. **定义图形命令：** Command List 中包含了一系列图形命令，如设置渲染目标、绑定顶点缓冲区、设置着色器资源等。
2. **提交到命令队列：** Command List 被提交到命令队列中，由命令队列负责将这些命令发送给GPU执行。
3. **异步执行：** 由于 Command List 的提交和执行是异步的，开发者可以在主线程中准备并提交下一帧的渲染命令，而GPU则在后台执行前一帧的命令。

在你提供的代码中，通过以下语句创建了一个 Direct3D 12 的 Command List：

```c
cppCopy code
ThrowIfFailed(m_device->CreateCommandList(0, D3D12_COMMAND_LIST_TYPE_DIRECT, m_commandAllocator.Get(), m_pipelineState.Get(), IID_PPV_ARGS(&m_commandList)));
```

其中，参数说明如下：

- `m_device`：Direct3D 12 设备对象。
- `D3D12_COMMAND_LIST_TYPE_DIRECT`：表示这是一个直接渲染的命令列表，用于执行图形渲染。
- `m_commandAllocator.Get()`：为命令列表提供命令分配器，命令分配器用于分配内存给命令列表，使其能够记录命令。
- `m_pipelineState.Get()`：为命令列表指定了使用的图形管线状态，即前面提到的 Pipeline State Object（PSO）。
- `IID_PPV_ARGS(&m_commandList)`：用于获取创建的 Command List 的接口指针。

**Command Queue 与 Command List 的区别：**

- **Command Queue：** 是用于管理和调度 Command List 的对象。Command Queue 接收和执行 Command List，确保它们按照正确的顺序和时间被提交给GPU。在DirectX 12中，通常有图形命令队列和计算命令队列等。
- **Command List：** 包含实际的图形或计算命令，定义了GPU要执行的具体操作。Command List 需要先创建，然后被提交到 Command Queue，由 Command Queue 管理它们的执行。

总体而言，Command Queue 是一个管理和执行 Command List 的调度者，而 Command List 包含了要执行的具体命令。这种设计使得开发者可以在不同线程中准备不同的 Command List，然后在主线程中将它们提交到 Command Queue 中，实现并行化和异步执行。

## vertex buffer

```
    // Create the vertex buffer.
    {
        // Define the geometry for a triangle.
        Vertex triangleVertices[] =
        {
            { { 0.0f, 0.25f * m_aspectRatio, 0.0f }, { 1.0f, 0.0f, 0.0f, 1.0f } },
            { { 0.25f, -0.25f * m_aspectRatio, 0.0f }, { 0.0f, 1.0f, 0.0f, 1.0f } },
            { { -0.25f, -0.25f * m_aspectRatio, 0.0f }, { 0.0f, 0.0f, 1.0f, 1.0f } }
        };

        const UINT vertexBufferSize = sizeof(triangleVertices);

        // Note: using upload heaps to transfer static data like vert buffers is not 
        // recommended. Every time the GPU needs it, the upload heap will be marshalled 
        // over. Please read up on Default Heap usage. An upload heap is used here for 
        // code simplicity and because there are very few verts to actually transfer.
        ThrowIfFailed(m_device->CreateCommittedResource(
            &CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_UPLOAD),
            D3D12_HEAP_FLAG_NONE,
            &CD3DX12_RESOURCE_DESC::Buffer(vertexBufferSize),
            D3D12_RESOURCE_STATE_GENERIC_READ,
            nullptr,
            IID_PPV_ARGS(&m_vertexBuffer)));

        // Copy the triangle data to the vertex buffer.
        UINT8* pVertexDataBegin;
        CD3DX12_RANGE readRange(0, 0);        // We do not intend to read from this resource on the CPU.
        ThrowIfFailed(m_vertexBuffer->Map(0, &readRange, reinterpret_cast<void**>(&pVertexDataBegin)));
        memcpy(pVertexDataBegin, triangleVertices, sizeof(triangleVertices));
        m_vertexBuffer->Unmap(0, nullptr);

        // Initialize the vertex buffer view.
        m_vertexBufferView.BufferLocation = m_vertexBuffer->GetGPUVirtualAddress();
        m_vertexBufferView.StrideInBytes = sizeof(Vertex);
        m_vertexBufferView.SizeInBytes = vertexBufferSize;
    }
```

这部分代码主要用于创建和初始化一个包含三角形几何数据的顶点缓冲区（Vertex Buffer）。以下是代码的解释：

```
cppCopy code// Define the geometry for a triangle.
Vertex triangleVertices[] =
{
    { { 0.0f, 0.25f * m_aspectRatio, 0.0f }, { 1.0f, 0.0f, 0.0f, 1.0f } },
    { { 0.25f, -0.25f * m_aspectRatio, 0.0f }, { 0.0f, 1.0f, 0.0f, 1.0f } },
    { { -0.25f, -0.25f * m_aspectRatio, 0.0f }, { 0.0f, 0.0f, 1.0f, 1.0f } }
};
```

在这里，定义了一个三角形的顶点数据，每个顶点包含一个位置（x、y、z坐标）和颜色（RGBA值）。

```
cppCopy code
const UINT vertexBufferSize = sizeof(triangleVertices);
```

计算了顶点缓冲区的大小。

```
cppCopy code// Note: using upload heaps to transfer static data like vert buffers is not 
// recommended. Every time the GPU needs it, the upload heap will be marshalled 
// over. Please read up on Default Heap usage. An upload heap is used here for 
// code simplicity and because there are very few verts to actually transfer.
```

这段注释说明了使用上传堆（Upload Heap）来传输静态数据（如顶点缓冲区数据）的一些注意事项。上传堆的数据在GPU需要时会被复制，因此对于大量的静态数据，推荐使用默认堆（Default Heap）来优化性能。但在这里，为了代码简单和因为顶点数量很少，选择使用上传堆。

```
cppCopy codeThrowIfFailed(m_device->CreateCommittedResource(
    &CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_UPLOAD),
    D3D12_HEAP_FLAG_NONE,
    &CD3DX12_RESOURCE_DESC::Buffer(vertexBufferSize),
    D3D12_RESOURCE_STATE_GENERIC_READ,
    nullptr,
    IID_PPV_ARGS(&m_vertexBuffer)));
```

这里使用 `CreateCommittedResource` 函数创建了一个上传堆，并为其分配了顶点缓冲区的内存。

```
cppCopy code// Copy the triangle data to the vertex buffer.
UINT8* pVertexDataBegin;
CD3DX12_RANGE readRange(0, 0);  // We do not intend to read from this resource on the CPU.
ThrowIfFailed(m_vertexBuffer->Map(0, &readRange, reinterpret_cast<void**>(&pVertexDataBegin)));
memcpy(pVertexDataBegin, triangleVertices, sizeof(triangleVertices));
m_vertexBuffer->Unmap(0, nullptr);
```

通过 `Map` 函数将顶点缓冲区映射到CPU的地址空间，然后使用 `memcpy` 将三角形的顶点数据拷贝到顶点缓冲区。最后使用 `Unmap` 函数解除映射。

```
cppCopy code// Initialize the vertex buffer view.
m_vertexBufferView.BufferLocation = m_vertexBuffer->GetGPUVirtualAddress();
m_vertexBufferView.StrideInBytes = sizeof(Vertex);
m_vertexBufferView.SizeInBytes = vertexBufferSize;
```

最后，初始化了顶点缓冲区视图（`D3D12_VERTEX_BUFFER_VIEW`）。这个视图告诉GPU如何访问顶点缓冲区。其中，`BufferLocation` 是顶点缓冲区的GPU虚拟地址，`StrideInBytes` 是每个顶点的字节大小，`SizeInBytes` 是整个顶点缓冲区的大小。这个视图将在渲染管线中被使用，确保正确地读取顶点数据。

## 同步对象

```c
// Create synchronization objects and wait until assets have been uploaded to the GPU.
{
    // 创建一个用于同步的 Fence 对象，初始值为 0。
    ThrowIfFailed(m_device->CreateFence(0, D3D12_FENCE_FLAG_NONE, IID_PPV_ARGS(&m_fence)));
    m_fenceValue = 1; // 初始 Fence 值设为 1。

    // 创建一个事件句柄，用于帧同步。
    m_fenceEvent = CreateEvent(nullptr, FALSE, FALSE, nullptr);
    if (m_fenceEvent == nullptr)
    {
        // 如果创建事件句柄失败，抛出异常。
        ThrowIfFailed(HRESULT_FROM_WIN32(GetLastError()));
    }

    // 等待命令列表执行完成；在主循环中我们会重用相同的命令列表，但现在我们只是希望在继续之前等待设置完成。
    WaitForPreviousFrame();
}
```

- **CreateFence：** 创建一个用于同步的 Fence 对象。Fence 是一种同步机制，用于标记GPU中的某个点，以确保CPU和GPU之间的同步。这里设置了初始值为 0，并且不使用特殊标志。
- **m_fenceValue：** Fence 的当前值，初始化为 1。
- **CreateEvent：** 创建一个事件句柄，用于帧同步。事件句柄是一种同步对象，可用于在不同线程之间进行信号传递。
- **WaitForPreviousFrame：** 等待前一帧的命令列表执行完成。这是一个函数调用，可能是在后面的代码中定义的。这个步骤的目的是确保在继续执行之前，前一帧的渲染或计算任务已经完成，以避免资源冲突。

这段代码的目的是在初始化过程中创建必要的同步对象，并等待前一帧的命令列表执行完成，以确保资源已经成功上传到GPU。这是一个常见的初始化模式，用于确保在应用程序主循环开始时，GPU已准备好渲染或执行计算任务。
