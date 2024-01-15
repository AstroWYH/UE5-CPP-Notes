```lua
-- 创建一个包含pair作为key的table
local myTable = {}

-- 定义一个pair
local myPair1 = { key = "x1", val = "y1" }
local myPair2 = { key = "x2", val = "y2" }

-- 将pair作为key插入table中（使用字符串作为键）
local keyString1 = myPair1.key .. "," .. myPair1.val
myTable[keyString1] = "A"
local keyString2 = myPair2.key .. "," .. myPair2.val
myTable[keyString2] = "B"

-- 查询时使用字符串作为key
local result1 = myTable["x1,y1"]
local result2 = myTable["x2,y2"]

-- 输出结果
print(result1)
print(result2)
```

[19:29:26] 虚拟机初始化完毕
[19:29:26] A
[19:29:26] B
[19:29:26] 虚拟机已停止运行
