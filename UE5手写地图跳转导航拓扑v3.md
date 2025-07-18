```cpp
// Copyright Epic Games, Inc. All Rights Reserved.

#include "BlankProgram.h"
#include "RequiredProgramMainCPPInclude.h"
#include "Containers/UnrealString.h"
#include "Containers/Ticker.h"
#include "Misc/OutputDeviceRedirector.h"
#include "Modules/ModuleManager.h"

// 定义日志分类
DEFINE_LOG_CATEGORY_STATIC(LogSmartPathFinder, Log, All);
// 声明程序入口
IMPLEMENT_APPLICATION(SmartPathFinderApp, "SmartPathFinderApp");

// 桥梁信息结构体：SrcMap → DstMap，经由传送点 TpTag
struct BridgeInfo
{
	FString SrcMap;   // 起点地图名称
	FString DstMap;   // 终点地图名称
	FString TpTag;    // 传送点标签（空字符串表示内部跳转）
};

// 路径段结构体：从 FromMap → ToMap，经由 ViaTpTag 跳转
struct RouteSegment
{
	FString FromMap;     // 路径起始地图
	FString ToMap;       // 路径目标地图
	FString ViaTpTag;    // 使用的传送点（空字符串表示内部跳转）
};

// 阶段目标结构体：最终消费的目标，包含地图和位置
struct StageGoal
{
	FString Map;   // 所在地图
	FString Pos;   // 目标位置，可为“(x,y)”坐标或“TP:xxx”格式
};

// 智能路径搜索器类：构建桥梁图并进行最短路径搜索
class FSmartPathFinder
{
public:
	// 构造函数，初始化邻接表
	FSmartPathFinder(const TArray<BridgeInfo>& InBridges)
	{
		for (const auto& E : InBridges)
		{
			Adj.FindOrAdd(E.SrcMap).Add(E);
		}
	}

	// 构建从 Start 到 Target 的最短路径（BFS），输出路径段数组
	bool BuildRoute(const FString& Start, const FString& Target, TArray<RouteSegment>& Out) const
	{
		Out.Empty();
		if (Start == Target)
		{
			Out.Add({ Start, Target, TEXT("") });
			return true;
		}

		TMap<FString, BridgeInfo> Prev;     // 用于记录路径的前向边
		TSet<FString> Visited;              // 已访问地图集合
		TQueue<FString> Q;                  // BFS 队列
		Visited.Add(Start);
		Q.Enqueue(Start);

		while (!Q.IsEmpty())
		{
			FString Cur;
			Q.Dequeue(Cur);

			// 查找当前地图所有可达的桥梁边
			if (const TArray<BridgeInfo>* Edges = Adj.Find(Cur))
			{
				for (const auto& E : *Edges)
				{
					if (!Visited.Contains(E.DstMap))
					{
						Visited.Add(E.DstMap);
						Q.Enqueue(E.DstMap);
						Prev.Add(E.DstMap, E);
						if (E.DstMap == Target) goto FOUND;
					}
				}
			}
		}
		return false;

FOUND:
		{
			TArray<RouteSegment> Rev;    // 反向构建路径段
			FString Cur = Target;
			while (Prev.Contains(Cur))
			{
				const auto& E = Prev[Cur];
				Rev.Add({ E.SrcMap, E.DstMap, E.TpTag });
				Cur = E.SrcMap;
			}
			Algo::Reverse(Rev);
			Out = MoveTemp(Rev);
			return true;
		}
	}

	// 将路径段转换为阶段目标（StageGoal），用于最终展示和消费
	TArray<StageGoal> ToStageGoals(const TArray<RouteSegment>& Segs, const FString& TargetMap, const FString& TargetPos) const
	{
		TArray<StageGoal> Goals;
		for (const auto& Seg : Segs)
		{
			if (!Seg.ViaTpTag.IsEmpty())
			{
				Goals.Add({ Seg.FromMap, FString::Printf(TEXT("%s"), *Seg.ViaTpTag) });
			}
		}
		Goals.Add({ TargetMap, TargetPos });
		return Goals;
	}

private:
	TMap<FString, TArray<BridgeInfo>> Adj; // 邻接表（每张地图可跳转的目标）
};

// 主程序入口点：用于运行测试用例
INT32_MAIN_INT32_ARGC_TCHAR_ARGV()
{
	GEngineLoop.PreInit(ArgC, ArgV);
	UE_LOG(LogSmartPathFinder, Log, TEXT("SmartPathFinder Console App Starting..."));

	// 设置桥梁连接关系：图结构 A→B→C 等
	TArray<BridgeInfo> Bridges = {
		{ TEXT("A"), TEXT("B"), TEXT("T1") },
		{ TEXT("B"), TEXT("C"), TEXT("T2") },
		{ TEXT("D"), TEXT("A"), TEXT("T3") },
	};

	FSmartPathFinder Nav(Bridges);

	// 测试用例结构体
	struct FTestCase
	{
		FString StartMap;
		FString TargetMap;
		FString TargetPos;
		FString Name;
	};

	// 示例测试路径
	TArray<FTestCase> Tests = {
		{ TEXT("A"), TEXT("A"), TEXT("(10,10)"), TEXT("A→A") },
		{ TEXT("A"), TEXT("B"), TEXT("(20,50)"), TEXT("A→B") },
		{ TEXT("A"), TEXT("C"), TEXT("(60,80)"), TEXT("A→C") },
		{ TEXT("D"), TEXT("C"), TEXT("(999,888)"), TEXT("D→C") }
	};

	// 遍历执行每个测试用例
	for (const auto& T : Tests)
	{
		UE_LOG(LogSmartPathFinder, Log, TEXT("\n=== 测试: %s ==="), *T.Name);
		UE_LOG(LogSmartPathFinder, Log, TEXT("%s -> %s 目标位置: %s"), *T.StartMap, *T.TargetMap, *T.TargetPos);

		TArray<RouteSegment> Segs;
		if (!Nav.BuildRoute(T.StartMap, T.TargetMap, Segs))
		{
			UE_LOG(LogSmartPathFinder, Warning, TEXT("无法找到路径"));
			continue;
		}

		// 输出每个路径段
		for (int32 i = 0; i < Segs.Num(); ++i)
		{
			const auto& S = Segs[i];
			UE_LOG(LogSmartPathFinder, Log, TEXT("  第 %d 步: %s → %s %s"), i + 1, *S.FromMap, *S.ToMap,
				S.ViaTpTag.IsEmpty() ? TEXT("(内部跳转)") : *FString::Printf(TEXT("via %s"), *S.ViaTpTag));
		}

		// 输出最终阶段目标列表
		auto Goals = Nav.ToStageGoals(Segs, T.TargetMap, T.TargetPos);
		for (int32 i = 0; i < Goals.Num(); ++i)
		{
			UE_LOG(LogSmartPathFinder, Log, TEXT("  阶段目标 %d: 地图=%s 去往位置=%s"), i + 1, *Goals[i].Map, *Goals[i].Pos);
		}
	}

	// 正常退出程序
	FEngineLoop::AppExit();
	return 0;
}
```
