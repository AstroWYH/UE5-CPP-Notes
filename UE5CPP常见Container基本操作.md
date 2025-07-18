```cpp
// Copyright Epic Games, Inc. All Rights Reserved.

#include "BlankProgram.h"

#include "CoreMinimal.h"
#include "Containers/Queue.h"
#include "Containers/Set.h"
#include "Containers/Map.h"
#include "Containers/Array.h"
#include "Containers/UnrealString.h"
#include "Containers/Ticker.h"
#include "Misc/OutputDeviceRedirector.h"
#include "Modules/ModuleManager.h"
#include "RequiredProgramMainCPPInclude.h"

IMPLEMENT_APPLICATION(Test, "Test");

DEFINE_LOG_CATEGORY_STATIC(LogDataStructureTest, Log, All);

class DataStructureTester
{
public:
	void Test();

private:
	void TestTArray();
	void TestTMap();
	void TestTSet();
	void TestTQueue();
	void TestTPair();
	void TestTLinkedList();
	void TestTMultiMap();
};

void DataStructureTester::Test()
{
	UE_LOG(LogDataStructureTest, Log, TEXT("=== Begin Data Structure Test ==="));

	TestTArray();
	TestTMap();
	TestTSet();
	TestTQueue();
	TestTPair();
	TestTLinkedList();
	TestTMultiMap();

	UE_LOG(LogDataStructureTest, Log, TEXT("=== End Data Structure Test ==="));
}

// --------------------- TArray ---------------------
void DataStructureTester::TestTArray()
{
	TArray<int32> Numbers;
	Numbers.Add(10);
	Numbers.Add(20);
	Numbers.Add(30);
	Numbers.Insert(15, 1);
	Numbers.Remove(20);
	Numbers.Emplace(40);

	UE_LOG(LogDataStructureTest, Log, TEXT("TArray 内容："));
	for (int32 Num : Numbers)
	{
		UE_LOG(LogDataStructureTest, Log, TEXT("%d"), Num);
	}

	Numbers[0] = 99;
	UE_LOG(LogDataStructureTest, Log, TEXT("第一个元素修改为：%d"), Numbers[0]);

	if (Numbers.IsValidIndex(2))
	{
		UE_LOG(LogDataStructureTest, Log, TEXT("索引 2 有效：%d"), Numbers[2]);
	}

	Numbers.Sort();
	UE_LOG(LogDataStructureTest, Log, TEXT("TArray 排序后："));
	for (int32 Num : Numbers)
	{
		UE_LOG(LogDataStructureTest, Log, TEXT("%d"), Num);
	}
}

// --------------------- TMap ---------------------
void DataStructureTester::TestTMap()
{
	TMap<FString, int32> ScoreMap;
	ScoreMap.Add(TEXT("Alice"), 90);
	ScoreMap.Add(TEXT("Bob"), 80);
	ScoreMap.Add(TEXT("Charlie"), 70);

	ScoreMap["Alice"] = 95;

	UE_LOG(LogDataStructureTest, Log, TEXT("TMap 内容："));
	for (const TPair<FString, int32>& Pair : ScoreMap)
	{
		UE_LOG(LogDataStructureTest, Log, TEXT("%s -> %d"), *Pair.Key, Pair.Value);
	}

	ScoreMap.Remove("Bob");
	UE_LOG(LogDataStructureTest, Log, TEXT("是否包含 Bob：%d"), ScoreMap.Contains("Bob"));

	int32* Found = ScoreMap.Find("Charlie");
	if (Found)
	{
		UE_LOG(LogDataStructureTest, Log, TEXT("Find Charlie: %d"), *Found);
	}

	int32& Ref = ScoreMap.FindOrAdd("Dave");
	Ref = 65;
	UE_LOG(LogDataStructureTest, Log, TEXT("Dave 分数：%d"), ScoreMap["Dave"]);

	TMap<int32, FString> IdToName;
	IdToName.Add(1, TEXT("One"));
	IdToName.Add(2, TEXT("Two"));

	// TryGet
	if (const FString* Result = IdToName.Find(2))
	{
		UE_LOG(LogDataStructureTest, Log, TEXT("Id 2 对应的是 %s"), **Result);
	}

	TArray<int32> Keys;
	IdToName.GetKeys(Keys);
	for (auto& Key : Keys)
	{
		UE_LOG(LogDataStructureTest, Log, TEXT("键：%d"), Key);
	}

	TArray<FString> Values;
	IdToName.GenerateValueArray(Values);
	for (auto& Val : Values)
	{
		UE_LOG(LogDataStructureTest, Log, TEXT("值：%s"), *Val);
	}
}

// --------------------- TSet ---------------------
void DataStructureTester::TestTSet()
{
	TSet<int32> UniqueNumbers;
	UniqueNumbers.Add(100);
	UniqueNumbers.Add(200);
	UniqueNumbers.Add(100);

	UE_LOG(LogDataStructureTest, Log, TEXT("TSet 内容："));
	for (int32 Val : UniqueNumbers)
	{
		UE_LOG(LogDataStructureTest, Log, TEXT("%d"), Val);
	}

	UniqueNumbers.Remove(200);
	UE_LOG(LogDataStructureTest, Log, TEXT("是否包含 200：%d"), UniqueNumbers.Contains(200));
}

// --------------------- TQueue ---------------------
void DataStructureTester::TestTQueue()
{
	TQueue<int32> Queue;
	Queue.Enqueue(1);
	Queue.Enqueue(2);
	Queue.Enqueue(3);

	int32 Value;
	UE_LOG(LogDataStructureTest, Log, TEXT("TQueue 出队内容："));
	while (Queue.Dequeue(Value))
	{
		UE_LOG(LogDataStructureTest, Log, TEXT("%d"), Value);
	}
}

// --------------------- TPair ---------------------
void DataStructureTester::TestTPair()
{
	TPair<FString, int32> Pair(TEXT("Test"), 123);
	UE_LOG(LogDataStructureTest, Log, TEXT("TPair: %s -> %d"), *Pair.Key, Pair.Value);
}

// --------------------- TLinkedList ---------------------
void DataStructureTester::TestTLinkedList()
{
	TDoubleLinkedList<int32> List;
	List.AddTail(1);
	List.AddTail(2);
	List.AddTail(3);

	UE_LOG(LogDataStructureTest, Log, TEXT("TDoubleLinkedList 内容："));
	for (TDoubleLinkedList<int32>::TIterator It(List.GetHead()); It; ++It)
	{
		UE_LOG(LogDataStructureTest, Log, TEXT("%d"), *It);
	}
}

// --------------------- TMultiMap ---------------------
void DataStructureTester::TestTMultiMap()
{
	TMultiMap<FString, int32> MultiMap;
	MultiMap.Add(TEXT("GroupA"), 10);
	MultiMap.Add(TEXT("GroupA"), 20);
	MultiMap.Add(TEXT("GroupB"), 30);

	UE_LOG(LogDataStructureTest, Log, TEXT("TMultiMap 内容："));
	for (auto& Pair : MultiMap)
	{
		UE_LOG(LogDataStructureTest, Log, TEXT("%s -> %d"), *Pair.Key, Pair.Value);
	}

	TArray<int32> Values;
	MultiMap.MultiFind(TEXT("GroupA"), Values);
	UE_LOG(LogDataStructureTest, Log, TEXT("GroupA 有 %d 个值"), Values.Num());
}

// 主程序入口点
INT32_MAIN_INT32_ARGC_TCHAR_ARGV()
{
	GEngineLoop.PreInit(ArgC, ArgV);
	UE_LOG(LogDataStructureTest, Log, TEXT("UE5 Container Test Starting..."));

	DataStructureTester Test;
	Test.Test();

	FEngineLoop::AppExit();
	return 0;
}
```
