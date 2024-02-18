```lua
-- 定义一个异步任务函数
function asyncTask(callback)
    -- 模拟异步操作，延迟 3 秒
    local function fakeAsyncOperation()
        for i = 1, 3 do
            print("Working...", i)
            -- 模拟异步操作，等待 1 秒
            coroutine.yield(1)
        end
        callback("Async task completed")
    end

    -- 创建协程并执行
    local co = coroutine.create(fakeAsyncOperation)
    coroutine.resume(co)
    return co  -- 返回协程对象
end

-- 模拟主循环，每帧调用一次协程
function mainLoop(co)  -- 接收协程对象作为参数
    -- 检查协程是否完成
    if coroutine.status(co) == "suspended" then
        -- 继续执行协程
        local success, waitTime = coroutine.resume(co)
        if success and waitTime then
            -- 模拟等待时间
            print("Waiting for", waitTime, "second(s)...")
        end
    else
        print("Async task done")
    end
end

-- 执行异步任务
local co = asyncTask(function(result)
    print(result)
end)

-- 模拟主循环，每帧调用 mainLoop
for i = 1, 10 do
    print("Frame", i)
    mainLoop(co)  -- 传递协程对象
end

[18:17:43] 虚拟机初始化完毕
[18:17:43] Working...	1
[18:17:43] Frame	1
[18:17:43] Working...	2
[18:17:43] Waiting for	1	second(s)...
[18:17:43] Frame	2
[18:17:43] Working...	3
[18:17:43] Waiting for	1	second(s)...
[18:17:43] Frame	3
[18:17:43] Async task completed
[18:17:43] Frame	4
[18:17:43] Async task done
[18:17:43] Frame	5
[18:17:43] Async task done
[18:17:43] Frame	6
[18:17:43] Async task done
[18:17:43] Frame	7
[18:17:43] Async task done
[18:17:43] Frame	8
[18:17:43] Async task done
[18:17:43] Frame	9
[18:17:43] Async task done
[18:17:43] Frame	10
[18:17:43] Async task done
[18:17:43] 虚拟机已停止运行
```

