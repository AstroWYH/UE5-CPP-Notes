```lua
-- A 类
A = {}

-- A 的构造函数
function A:new(name)
    local instance = {name = name}
    setmetatable(instance, self)
    self.__index = self
    return instance
end

-- A 类的函数（这个函数将作为委托传递给 B）
function A:onEventTriggered(message)
    print(self.name .. " received event: " .. message)
end

-- B 类
B = {}

-- B 的构造函数
function B:new()
    local instance = {delegates = {}}
    setmetatable(instance, self)
    self.__index = self
    return instance
end

-- B 类的函数（接收委托并调用）
function B:registerDelegate(callback)
    table.insert(self.delegates, callback)
end

function B:triggerEvent(message)
    -- B 在某个事件发生时调用所有注册的委托
    for _, delegate in ipairs(self.delegates) do
        delegate(message)
    end
end

-- 创建 A 和 B 的实例
local a1 = A:new("Player1")
local a2 = A:new("Player2")
local b = B:new()

-- 将 A 的函数作为委托注册到 B 中
b:registerDelegate(function(message) a1:onEventTriggered(message) end)
b:registerDelegate(function(message) a2:onEventTriggered(message) end)

-- 当 B 触发事件时，A 的函数被调用
b:triggerEvent("Level Up!")  -- 输出：Player1 received event: Level Up!
                              -- 输出：Player2 received event: Level Up!
```

[17:07:46] 虚拟机初始化完毕
[17:07:46] Player1 received event: Level Up!
[17:07:46] Player2 received event: Level Up!
[17:07:46] 虚拟机已停止运行