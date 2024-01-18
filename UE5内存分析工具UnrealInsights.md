![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/afcaff1c-8f07-4046-9d6f-65d850e71850)
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/4c029cbb-e408-4ee6-89d1-5b2328a90327)
每次跳跃增加20M内存，可以很直观的看见。

1 选用DebugGame Editor，设置以下属性。直接以游戏-game启动，并设置相应的trace配置条件。
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/a2dabddd-9e1f-4e38-8f51-f8cf97bd7c6d)
2 找到对应UE版本的UnrealInsights.exe
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/d5f383f6-c472-4fe0-9d98-ad8eb50af892)
3 VS启动游戏，观察UnrealInsights新增的Live，点击OpenTrace。可见内存的使用情况，包括一系列LLM内存的使用详情。
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/059340d4-9dc0-41d7-a535-78efb1ee5225)

LLM Tags
可以把想列表里想查看内存的对象类在左边的视图列出来
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/b94f3ac2-cbe1-42a6-b6fb-a77a4471f74e)
 Investigation
内存调查系统, 可以使用规则(大于某个时间点，小于某个时间点，在某两个时间点之间)来显示一段时间内分配而一直未释放的内存, 下面是查看从10秒 ~ 20s的内存变化(10s开始分配并且到20s的时候还未释放的内存统计
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/166f24e9-16fb-4523-9b4f-1f77358aa015)
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/8c437c40-f7b9-4438-83b7-022d34b79d28)
 这里点击Hierarchy每个alloc名字左边可以查看分配程序栈，内存大小，源文件等等:
 ![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/2ec81e95-9768-4b2f-a89e-607c5f4a8faf)
 ![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/f7fe2165-57f4-4f18-997d-ad112d0943ef)
看了下感觉美中不足的是，缺乏信息，像Memort -full指令能指出一个object对象的具体名字和地图路径。
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/835012a2-595e-4be3-81fc-2afce5c01395)

部分参考：https://blog.csdn.net/qq_29523119/article/details/131628733
