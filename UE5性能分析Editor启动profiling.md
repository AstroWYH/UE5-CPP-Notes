# 1 耗时统计

**开启trace**

start "" "%~dp0\Engine\Binaries\Win64\UnrealEditor" "%~dp0\xxx\xxx.uproject" -WINDOWED -trace=default,bookmark,loadtime,file -log

![img](Images/UE5性能分析Editor启动profiling/image2025-7-28_15-0-48.png)

**trace文件（可下载）：![img](Images/UE5性能分析Editor启动profiling/placeholdertype=unknown&name=20250728_145658.rar&attachmentId=53381952&version=1&mimeType=application%2Foctet-stream&height=150)**

**开启trace**

[2025.07.28-06.57.43:561][  0]LogUnrealEdMisc: Loading editor; pre map load, took 45.633

[2025.07.28-06.58.37:946][  0]LogUnrealEdMisc: Total Editor Startup Time, took 100.018

**不开trace**

[2025.07.28-06.50.15:954][  0]LogUnrealEdMisc: Loading editor; pre map load, took 47.027

[2025.07.28-06.51.08:401][  0]LogUnrealEdMisc: Total Editor Startup Time, took 99.473



**结论：**开启trace的情况下启动editor，从log和insights结合看，时间大概在100s+，有时会更长。

不开trace，时间90s+，多次启动平均差不多都是这个值，差距不大，这个是实际启动editor的真实时间。

# 2 trace分析

![新诛仙项目 > Editor启动profiling > image2025-7-28_15-0-48.png](Images/UE5性能分析Editor启动profiling/image2025-7-28_15-0-48.png)

从trace文件可看出，耗时分为几个部分：

## 1 FEngineLoop::PreInitPreStartupScreen (9s)

![新诛仙项目 > Editor启动profiling > image2025-7-28_15-9-21.png](Images/UE5性能分析Editor启动profiling/image2025-7-28_15-9-21.png)

这部分看着没什么优化空间。

## 2 FEngineLoop::PreInitPostStartupScreen (26s)

![新诛仙项目 > Editor启动profiling > image2025-7-28_15-10-48.png](Images/UE5性能分析Editor启动profiling/image2025-7-28_15-10-48.png)

这部分仔细看了下，也不存在优化空间。

## 3 EditorInit (1m 12s)

![新诛仙项目 > Editor启动profiling > image2025-7-28_15-14-24.png](Images/UE5性能分析Editor启动profiling/image2025-7-28_15-14-24.png)

直接看最后这个加载的部分，1m12s。

![新诛仙项目 > Editor启动profiling > image2025-7-28_15-17-0.png](Images/UE5性能分析Editor启动profiling/image2025-7-28_15-17-0.png)

## 问题1：这15s在干什么？

![新诛仙项目 > Editor启动profiling > image2025-7-28_15-19-41.png](Images/UE5性能分析Editor启动profiling/image2025-7-28_15-19-41.png)

## 问题2：这35s是重点。

![新诛仙项目 > Editor启动profiling > image2025-7-28_15-20-47.png](Images/UE5性能分析Editor启动profiling/image2025-7-28_15-20-47.png)

![新诛仙项目 > Editor启动profiling > image2025-7-28_15-21-18.png](Images/UE5性能分析Editor启动profiling/image2025-7-28_15-21-18.png)

FPackageName::DoesPackageExistEx (42.7 µs)

FPackageName::SearchForPackageOnDisk (27 µs)

放大后，全是密密麻麻的资源查找、检查确认相关的操作。