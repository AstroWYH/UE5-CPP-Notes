```
function doSomethingAsync(param, callback)
    print("Doing something with param:", param)
    local result = (param or 4) * 2
    -- 检查是否有回调函数
    if callback then
        callback(result)  -- 调用回调函数
    else
        print("No callback provided, result:", result)
    end
end

-- 调用时传递回调
doSomethingAsync(10, function(result)
    print("Callback called with result:", result)
end)

-- 调用时不传递回调
doSomethingAsync(5)
doSomethingAsync()

```

[11:25:34] 虚拟机初始化完毕
[11:25:34] Doing something with param:	10
[11:25:34] Callback called with result:	20
[11:25:34] Doing something with param:	5
[11:25:34] No callback provided, result:	10
[11:25:34] Doing something with param:	nil
[11:25:34] No callback provided, result:	8
[11:25:34] 虚拟机已停止运行