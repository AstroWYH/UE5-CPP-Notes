## Run

```c
int Win32Application::Run(DXSample* pSample, HINSTANCE hInstance, int nCmdShow)
{
    // Parse the command line parameters
    int argc;
    LPWSTR* argv = CommandLineToArgvW(GetCommandLineW(), &argc);
    pSample->ParseCommandLineArgs(argv, argc);
    LocalFree(argv);

    // Initialize the window class.
    WNDCLASSEX windowClass = { 0 };
    windowClass.cbSize = sizeof(WNDCLASSEX);
    windowClass.style = CS_HREDRAW | CS_VREDRAW;
    windowClass.lpfnWndProc = WindowProc;
    windowClass.hInstance = hInstance;
    windowClass.hCursor = LoadCursor(NULL, IDC_ARROW);
    windowClass.lpszClassName = L"DXSampleClass";
    RegisterClassEx(&windowClass);

    RECT windowRect = { 0, 0, static_cast<LONG>(pSample->GetWidth()), static_cast<LONG>(pSample->GetHeight()) };
    AdjustWindowRect(&windowRect, WS_OVERLAPPEDWINDOW, FALSE);

    // Create the window and store a handle to it.
    m_hwnd = CreateWindow(
        windowClass.lpszClassName,
        pSample->GetTitle(),
        WS_OVERLAPPEDWINDOW,
        CW_USEDEFAULT,
        CW_USEDEFAULT,
        windowRect.right - windowRect.left,
        windowRect.bottom - windowRect.top,
        nullptr,        // We have no parent window.
        nullptr,        // We aren't using menus.
        hInstance,
        pSample);

    // Initialize the sample. OnInit is defined in each child-implementation of DXSample.
    pSample->OnInit();

    ShowWindow(m_hwnd, nCmdShow);

    // Main sample loop.
    MSG msg = {};
    while (msg.message != WM_QUIT)
    {
        // Process any messages in the queue.
        if (PeekMessage(&msg, NULL, 0, 0, PM_REMOVE))
        {
            TranslateMessage(&msg);
            DispatchMessage(&msg);
        }
    }

    pSample->OnDestroy();

    // Return this part of the WM_QUIT message to Windows.
    return static_cast<char>(msg.wParam);
}
```

这段代码是一个 Win32 应用程序的运行主循环，用于初始化窗口、执行消息循环和资源清理。以下是对代码的详细解释：

- **`CommandLineToArgvW`：** 用于解析命令行参数，获取 argc 和 argv。这些参数在 `pSample->ParseCommandLineArgs` 中会被用于解析应用程序的命令行参数。
- **`RegisterClassEx`：** 注册窗口类，定义窗口的属性，包括窗口过程（`WindowProc`）、实例句柄（`hInstance`）、光标等。
- **`AdjustWindowRect`：** 调整窗口矩形，确保窗口的客户区域大小符合期望。
- **`CreateWindow`：** 创建窗口，并存储窗口句柄（`m_hwnd`）。这里指定了窗口类名、窗口标题、样式等信息。
- **`pSample->OnInit()`：** 调用示例程序的初始化函数，该函数通常包含DirectX 12的初始化步骤，例如创建设备、交换链、资源等。
- **`ShowWindow`：** 显示窗口。
- **主循环（`while (msg.message != WM_QUIT)`）：** 进入消息循环，等待系统消息。循环条件是直到收到退出消息（`WM_QUIT`）。
- **消息处理：** 使用 `PeekMessage` 函数检查消息队列，如果有消息则将消息从队列中取出，然后使用 `TranslateMessage` 和 `DispatchMessage` 处理消息。
- **`pSample->OnDestroy()`：** 在退出循环后，调用示例程序的销毁函数，该函数通常包含了释放DirectX 12资源、清理内存等工作。
- **返回退出消息的 wParam：** 返回退出消息的 wParam，通常为程序的返回码。

整个代码片段描述了一个完整的 Win32 应用程序主循环，其中包含了初始化、消息处理和资源清理等步骤。

## Render

```c
// Render the scene.
void D3D12HelloTriangle::OnRender()
{
    // Record all the commands we need to render the scene into the command list.
    PopulateCommandList();

    // Execute the command list.
    ID3D12CommandList* ppCommandLists[] = { m_commandList.Get() };
    m_commandQueue->ExecuteCommandLists(_countof(ppCommandLists), ppCommandLists);

    // Present the frame.
    ThrowIfFailed(m_swapChain->Present(1, 0));

    WaitForPreviousFrame();
}
```

- **`PopulateCommandList`：** 这个函数的目的是记录所有渲染场景所需的命令到命令列表 (`m_commandList`) 中。这些命令可能包括设置渲染管线状态、绑定顶点缓冲区、设置着色器资源等。在实际应用中，这个函数会包含渲染场景所需的所有绘制和设置命令。
- **`ExecuteCommandLists`：** 这里调用了 `m_commandQueue->ExecuteCommandLists` 来执行命令列表。`m_commandQueue` 是一个命令队列，它负责接收并执行提交到它的命令列表。在这个函数中，只有一个命令列表 (`m_commandList`) 被执行。
- **`m_swapChain->Present`：** 这个函数用于呈现帧。在基于交换链的应用程序中，`Present` 函数将当前的后缓冲区的内容呈现到屏幕上。参数 `1` 表示垂直同步（V-Sync）等待一个垂直同步信号，参数 `0` 表示不等待。这个函数的调用实际上将渲染好的图像显示在屏幕上。
- **`WaitForPreviousFrame`：** 这个函数用于等待前一帧的渲染完成。在 D3D12 中，命令队列是异步执行的，因此需要确保前一帧的命令执行完毕后再提交新的帧。这样做可以防止资源冲突和同步问题。`WaitForPreviousFrame` 通常会等待前一帧的帧同步信号，确保前一帧的命令都已经执行完毕。

整体来说，`OnRender` 函数包含了渲染一个帧所需的主要步骤，包括记录命令、执行命令、呈现帧以及等待前一帧的完成。在实际应用中，`PopulateCommandList` 函数会根据具体的场景和需求生成相应的渲染命令。这个函数可能会在每一帧都被调用，以确保更新渲染场景。

## PopulateCommandList

```c
void D3D12HelloTriangle::PopulateCommandList()
{
    // 命令列表分配器只能在与关联的命令列表在GPU上执行完成后重置；
    // 应用程序应该使用fences来确定GPU执行的进度。
    ThrowIfFailed(m_commandAllocator->Reset());

    // 然而，当在特定命令列表上调用ExecuteCommandList()时，
    // 该命令列表可以在任何时候被重置，并且在重新记录之前必须被重置。
    ThrowIfFailed(m_commandList->Reset(m_commandAllocator.Get(), m_pipelineState.Get()));

    // 设置必要的状态。
    m_commandList->SetGraphicsRootSignature(m_rootSignature.Get());
    m_commandList->RSSetViewports(1, &m_viewport);
    m_commandList->RSSetScissorRects(1, &m_scissorRect);

    // 表示后缓冲区将被用作渲染目标。
    m_commandList->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_renderTargets[m_frameIndex].Get(), D3D12_RESOURCE_STATE_PRESENT, D3D12_RESOURCE_STATE_RENDER_TARGET));

    // 获取当前后缓冲区的描述符句柄。
    CD3DX12_CPU_DESCRIPTOR_HANDLE rtvHandle(m_rtvHeap->GetCPUDescriptorHandleForHeapStart(), m_frameIndex, m_rtvDescriptorSize);
    m_commandList->OMSetRenderTargets(1, &rtvHandle, FALSE, nullptr);

    // 记录渲染命令。
    const float clearColor[] = { 0.0f, 0.2f, 0.4f, 1.0f };
    m_commandList->ClearRenderTargetView(rtvHandle, clearColor, 0, nullptr);
    m_commandList->IASetPrimitiveTopology(D3D_PRIMITIVE_TOPOLOGY_TRIANGLELIST);
    m_commandList->IASetVertexBuffers(0, 1, &m_vertexBufferView);
    m_commandList->DrawInstanced(3, 1, 0, 0);

    // 表示后缓冲区现在将被用于呈现。
    m_commandList->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_renderTargets[m_frameIndex].Get(), D3D12_RESOURCE_STATE_RENDER_TARGET, D3D12_RESOURCE_STATE_PRESENT));

    ThrowIfFailed(m_commandList->Close());
}
```

- **`m_commandAllocator->Reset()`：** 重置命令列表分配器，确保它可以被重新使用。

- **`m_commandList->Reset(m_commandAllocator.Get(), m_pipelineState.Get())`：** 重置命令列表，使其准备好接受新的命令。这里使用了之前创建的命令列表分配器和渲染管线状态。

- **设置必要的状态：**

  - **`m_commandList->SetGraphicsRootSignature(m_rootSignature.Get())`：** 设置图形根签名，指定渲染使用的根签名。
  - **`m_commandList->RSSetViewports(1, &m_viewport)` 和 `m_commandList->RSSetScissorRects(1, &m_scissorRect)`：** 设置视口和裁剪矩形。

- **`m_commandList->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(...))`：** 在使用后缓冲区作为渲染目标之前，需要进行资源状态转换。这里表示从呈现状态切换到渲染目标状态。

- **`m_commandList->OMSetRenderTargets(1, &rtvHandle, FALSE, nullptr)`：** 设置渲染目标，指定后缓冲区作为渲染目标。

- **记录渲染命令：**

  - **`m_commandList->ClearRenderTargetView(rtvHandle, clearColor, 0, nullptr)`：** 清除渲染目标视图，使用指定的清除颜色。
  - **`m_commandList->IASetPrimitiveTopology(D3D_PRIMITIVE_TOPOLOGY_TRIANGLELIST)`：** 设置图元拓扑类型为三角形列表。
  - **`m_commandList->IASetVertexBuffers(0, 1, &m_vertexBufferView)`：** 设置顶点缓冲区。

- **`m_commandList->DrawInstanced(3, 1, 0, 0)`：** 绘制三个顶点，形成一个三角形。

- **`m_commandList->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(...))`：** 在完成渲染后，需要将后缓冲区的状态从渲染目标切换回呈现状态。

- **`m_commandList->Close()`：** 完成命令列表的记录，关闭命令列表，确保它可以被提交执行。

  整个 `PopulateCommandList` 函数的流程是为当前帧填充命令列表，这些命令用于绘制一个清除颜色的三角形。以下是详细的步骤：

  1. **重置命令列表分配器：** 通过 `m_commandAllocator->Reset()` 来重置命令列表分配器，以确保它可以被重新使用。
  2. **重置命令列表：** 通过 `m_commandList->Reset(m_commandAllocator.Get(), m_pipelineState.Get())` 重置命令列表，准备接受新的命令。这里使用之前创建的命令列表分配器和渲染管线状态。
  3. **设置必要的状态：**
     - 设置图形根签名：`m_commandList->SetGraphicsRootSignature(m_rootSignature.Get())`，指定渲染使用的根签名。
     - 设置视口和裁剪矩形：`m_commandList->RSSetViewports(1, &m_viewport)` 和 `m_commandList->RSSetScissorRects(1, &m_scissorRect)`。
  4. **资源状态转换：** 通过 `m_commandList->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(...))` 完成从呈现状态到渲染目标状态的资源状态转换。
  5. **设置渲染目标：** 通过 `m_commandList->OMSetRenderTargets(1, &rtvHandle, FALSE, nullptr)` 设置渲染目标，指定后缓冲区作为渲染目标。
  6. **记录渲染命令：**
     - 清除渲染目标视图：`m_commandList->ClearRenderTargetView(rtvHandle, clearColor, 0, nullptr)`，使用指定的清除颜色。
     - 设置图元拓扑类型为三角形列表：`m_commandList->IASetPrimitiveTopology(D3D_PRIMITIVE_TOPOLOGY_TRIANGLELIST)`。
     - 设置顶点缓冲区：`m_commandList->IASetVertexBuffers(0, 1, &m_vertexBufferView)`。
     - 绘制三个顶点，形成一个三角形：`m_commandList->DrawInstanced(3, 1, 0, 0)`。
  7. **资源状态转换：** 通过 `m_commandList->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(...))` 完成从渲染目标状态到呈现状态的资源状态转换。
  8. **关闭命令列表：** 通过 `m_commandList->Close()` 完成命令列表的记录，确保它可以被提交执行。

  这样，`PopulateCommandList` 函数就准备好了当前帧的命令列表，用于渲染一个带有清除颜色的三角形。在渲染循环中，这个命令列表将被提交到命令队列，然后在 GPU 上执行。

## D3D12HelloTriangle

```c
class D3D12HelloTriangle : public DXSample
{
public:
    D3D12HelloTriangle(UINT width, UINT height, std::wstring name);

    virtual void OnInit();
    virtual void OnUpdate();
    virtual void OnRender();
    virtual void OnDestroy();

private:
    static const UINT FrameCount = 2;

    struct Vertex
    {
        XMFLOAT3 position;
        XMFLOAT4 color;
    };

    // Pipeline objects.
    CD3DX12_VIEWPORT m_viewport;
    CD3DX12_RECT m_scissorRect;
    ComPtr<IDXGISwapChain3> m_swapChain;
    ComPtr<ID3D12Device> m_device;
    ComPtr<ID3D12Resource> m_renderTargets[FrameCount];
    ComPtr<ID3D12CommandAllocator> m_commandAllocator;
    ComPtr<ID3D12CommandQueue> m_commandQueue;
    ComPtr<ID3D12RootSignature> m_rootSignature;
    ComPtr<ID3D12DescriptorHeap> m_rtvHeap;
    ComPtr<ID3D12PipelineState> m_pipelineState;
    ComPtr<ID3D12GraphicsCommandList> m_commandList;
    UINT m_rtvDescriptorSize;

    // App resources.
    ComPtr<ID3D12Resource> m_vertexBuffer;
    D3D12_VERTEX_BUFFER_VIEW m_vertexBufferView;

    // Synchronization objects.
    UINT m_frameIndex;
    HANDLE m_fenceEvent;
    ComPtr<ID3D12Fence> m_fence;
    UINT64 m_fenceValue;

    void LoadPipeline();
    void LoadAssets();
    void PopulateCommandList();
    void WaitForPreviousFrame();
};
```

## DXSample

```c
class DXSample
{
public:
    DXSample(UINT width, UINT height, std::wstring name);
    virtual ~DXSample();

    virtual void OnInit() = 0;
    virtual void OnUpdate() = 0;
    virtual void OnRender() = 0;
    virtual void OnDestroy() = 0;

    // Samples override the event handlers to handle specific messages.
    virtual void OnKeyDown(UINT8 /*key*/)   {}
    virtual void OnKeyUp(UINT8 /*key*/)     {}

    // Accessors.
    UINT GetWidth() const           { return m_width; }
    UINT GetHeight() const          { return m_height; }
    const WCHAR* GetTitle() const   { return m_title.c_str(); }

    void ParseCommandLineArgs(_In_reads_(argc) WCHAR* argv[], int argc);

protected:
    std::wstring GetAssetFullPath(LPCWSTR assetName);

    void GetHardwareAdapter(
        _In_ IDXGIFactory1* pFactory,
        _Outptr_result_maybenull_ IDXGIAdapter1** ppAdapter,
        bool requestHighPerformanceAdapter = false);

    void SetCustomWindowText(LPCWSTR text);

    // Viewport dimensions.
    UINT m_width;
    UINT m_height;
    float m_aspectRatio;

    // Adapter info.
    bool m_useWarpDevice;

private:
    // Root assets path.
    std::wstring m_assetsPath;

    // Window title.
    std::wstring m_title;
};

```
