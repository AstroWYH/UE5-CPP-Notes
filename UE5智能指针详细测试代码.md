```cpp
#pragma once

#include "CoreMinimal.h"
#include "UObject/ObjectMacros.h"
#include "UObject/Object.h"
#include "SmartPointerTester.generated.h"

class FSmartObject
{
public:
	void PrintInfo(const FString& Prefix) const
	{
		UE_LOG(LogTemp, Log, TEXT("%s: FSmartObject at %p"), *Prefix, this);
	}
};

UCLASS()
class USmartObject : public UObject
{
	GENERATED_BODY()

public:
	void PrintInfo(const FString& PointerType) const
	{
		UE_LOG(LogTemp, Log, TEXT("%s pointing to USmartObject with Hash: %d"), *PointerType, GetTypeHash(this));
	}
};

// 智能指针测试类
UCLASS()
class USmartPointerTester : public UObject
{
	GENERATED_BODY()

public:
	// 测试所有智能指针
	void TestAllSmartPointers();

private:
	// 各智能指针测试方法
	void TestTSharedPtr();
	void TestTSharedRef();
	void TestTWeakPtr();
	void TestTUniquePtr();
	void TestTObjectPtr(bool bMarkAsGarbage);
	void TestTStrongObjectPtr(bool bMarkAsGarbage);
	void TestTWeakObjectPtr(bool bMarkAsGarbage);
	void TestTRefCountPtr();

	// 辅助方法
	void SimulateGarbageCollection();
	void PrintValidity(const FString& Label, USmartObject* Object);
};

// Copyright Night Gamer, Inc. All Rights Reserved.

#include "SmartPointerTester.h"

#include "Modules/ModuleManager.h"
#include "UObject/StrongObjectPtr.h"
#include "UObject/WeakObjectPtr.h"
#include "Templates/SharedPointer.h"
#include "Templates/UniquePtr.h"
#include "HAL/PlatformProcess.h"

void USmartPointerTester::TestAllSmartPointers()
{
	UE_LOG(LogTemp, Log, TEXT("===== Starting Smart Pointer Tests ====="));

	TestTSharedPtr();
	TestTSharedRef();
	TestTWeakPtr();
	TestTUniquePtr();
	TestTRefCountPtr();

	TestTObjectPtr(false);
	TestTObjectPtr(true);
	TestTStrongObjectPtr(false);
	TestTStrongObjectPtr(true);
	TestTWeakObjectPtr(false);
	TestTWeakObjectPtr(true);

	UE_LOG(LogTemp, Log, TEXT("===== All Smart Pointer Tests Completed ====="));
}

void USmartPointerTester::TestTSharedPtr()
{
	UE_LOG(LogTemp, Log, TEXT("\n=== Testing TSharedPtr ==="));

	TSharedPtr<FSmartObject> SharedPtr = MakeShared<FSmartObject>();
	if (SharedPtr.IsValid())
	{
		SharedPtr->PrintInfo("TSharedPtr");

		TSharedPtr<FSmartObject> CopiedPtr = SharedPtr;
		CopiedPtr->PrintInfo("Copied TSharedPtr");

		UE_LOG(LogTemp, Log, TEXT("TSharedPtr RefCount: %d"), SharedPtr.GetSharedReferenceCount());
	}
}

void USmartPointerTester::TestTSharedRef()
{
	UE_LOG(LogTemp, Log, TEXT("\n=== Testing TSharedRef ==="));

	TSharedRef<FSmartObject> SharedRef = MakeShared<FSmartObject>();
	SharedRef->PrintInfo("TSharedRef");

	TSharedRef<FSmartObject> CopiedRef = SharedRef;
	CopiedRef->PrintInfo("Copied TSharedRef");

	UE_LOG(LogTemp, Log, TEXT("TSharedRef RefCount: %d"), SharedRef.GetSharedReferenceCount());
}

void USmartPointerTester::TestTWeakPtr()
{
	UE_LOG(LogTemp, Log, TEXT("\n=== Testing TWeakPtr ==="));

	TSharedPtr<FSmartObject> SharedPtr = MakeShared<FSmartObject>();
	TWeakPtr<FSmartObject> WeakPtr = SharedPtr;

	if (TSharedPtr<FSmartObject> LockedPtr = WeakPtr.Pin())
	{
		LockedPtr->PrintInfo("Locked TWeakPtr");
	}

	SharedPtr.Reset();

	if (TSharedPtr<FSmartObject> LockedPtr = WeakPtr.Pin())
	{
		LockedPtr->PrintInfo("Locked TWeakPtr (after reset)");
	}
	else
	{
		UE_LOG(LogTemp, Warning, TEXT("TWeakPtr is invalid after reset!"));
	}
}

void USmartPointerTester::TestTUniquePtr()
{
	UE_LOG(LogTemp, Log, TEXT("\n=== Testing TUniquePtr ==="));

	TUniquePtr<FSmartObject> UniquePtr = MakeUnique<FSmartObject>();
	if (UniquePtr.IsValid())
	{
		UniquePtr->PrintInfo("TUniquePtr");

		TUniquePtr<FSmartObject> MovedPtr = MoveTemp(UniquePtr);
		if (MovedPtr.IsValid())
		{
			MovedPtr->PrintInfo("Moved TUniquePtr");
		}

		if (!UniquePtr.IsValid())
		{
			UE_LOG(LogTemp, Log, TEXT("Original TUniquePtr is invalid after move"));
		}
	}
}

void USmartPointerTester::TestTRefCountPtr()
{
	UE_LOG(LogTemp, Log, TEXT("\n=== Testing TRefCountPtr ==="));

	class FTestRefCountedObject : public FRefCountedObject
	{
	public:
		void PrintInfo(const FString& Prefix) const
		{
			UE_LOG(LogTemp, Log, TEXT("%s: FTestRefCountedObject (RefCount: %d)"), *Prefix, GetRefCount());
		}
	};

	TRefCountPtr<FTestRefCountedObject> RefPtr = new FTestRefCountedObject();
	if (RefPtr.IsValid())
	{
		RefPtr->PrintInfo("TRefCountPtr");

		TRefCountPtr<FTestRefCountedObject> CopiedPtr = RefPtr;
		CopiedPtr->PrintInfo("Copied TRefCountPtr");

		UE_LOG(LogTemp, Log, TEXT("TRefCountPtr RefCount: %d"), RefPtr->GetRefCount());
	}
}

void USmartPointerTester::TestTObjectPtr(bool bMarkAsGarbage)
{
	UE_LOG(LogTemp, Warning, TEXT("\n=== [TObjectPtr] Test: %s ==="), bMarkAsGarbage ? TEXT("With MarkAsGarbage") : TEXT("Without MarkAsGarbage"));

	TObjectPtr<USmartObject> ObjPtr = NewObject<USmartObject>();
	PrintValidity(TEXT("Before GC"), ObjPtr);

	if (bMarkAsGarbage)
	{
		ObjPtr->MarkAsGarbage();
	}

	SimulateGarbageCollection();
	PrintValidity(TEXT("After GC"), ObjPtr);
}

void USmartPointerTester::TestTStrongObjectPtr(bool bMarkAsGarbage)
{
	UE_LOG(LogTemp, Warning, TEXT("\n=== [TStrongObjectPtr] Test: %s ==="), bMarkAsGarbage ? TEXT("With MarkAsGarbage") : TEXT("Without MarkAsGarbage"));

	TStrongObjectPtr<USmartObject> StrongPtr(NewObject<USmartObject>());
	PrintValidity(TEXT("Before GC"), StrongPtr.Get());

	if (bMarkAsGarbage)
	{
		StrongPtr->MarkAsGarbage();
	}

	SimulateGarbageCollection();
	PrintValidity(TEXT("After GC"), StrongPtr.Get());
}

void USmartPointerTester::TestTWeakObjectPtr(bool bMarkAsGarbage)
{
	UE_LOG(LogTemp, Warning, TEXT("\n=== [TWeakObjectPtr] Test: %s ==="), bMarkAsGarbage ? TEXT("With MarkAsGarbage") : TEXT("Without MarkAsGarbage"));

	USmartObject* RawObj = NewObject<USmartObject>();
	TWeakObjectPtr<USmartObject> WeakPtr(RawObj);
	PrintValidity(TEXT("Before GC"), WeakPtr.Get());

	if (bMarkAsGarbage)
	{
		RawObj->MarkAsGarbage();
	}

	SimulateGarbageCollection();
	PrintValidity(TEXT("After GC"), WeakPtr.Get());
}

void USmartPointerTester::SimulateGarbageCollection()
{
	UE_LOG(LogTemp, Log, TEXT("Simulating garbage collection..."));
	CollectGarbage(GARBAGE_COLLECTION_KEEPFLAGS);
	FPlatformProcess::Sleep(0.1f);
}

void USmartPointerTester::PrintValidity(const FString& Label, USmartObject* Object)
{
	if (IsValid(Object))
	{
		UE_LOG(LogTemp, Warning, TEXT("%s: Object is still valid."), *Label);
		Object->PrintInfo(*Label);
	}
	else
	{
		UE_LOG(LogTemp, Warning, TEXT("%s: Object is invalid or GC'd."), *Label);
	}
}
  

```
