```cpp
// Copyright Night Gamer, Inc. All Rights Reserved.

#include "BlankProgram.h"

#include "CoreMinimal.h"
#include "Modules/ModuleManager.h"
#include "RequiredProgramMainCPPInclude.h"
#include "UObject/StrongObjectPtr.h"
#include "UObject/WeakObjectPtr.h"
#include "Templates/SharedPointer.h"
#include "Templates/UniquePtr.h"
#include "HAL/PlatformProcess.h"
#include "UObject/Package.h"

void USmartPointerTester::TestAllSmartPointers()
{
	UE_LOG(LogTemp, Log, TEXT("===== Starting Smart Pointer Tests ====="));

	TestTSharedPtr();
	TestTSharedRef();
	TestTWeakPtr();
	TestTUniquePtr();
	TestTStrongObjectPtr();
	TestTWeakObjectPtr();
	TestTRefCountPtr();

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

void USmartPointerTester::TestTStrongObjectPtr()
{
	UE_LOG(LogTemp, Log, TEXT("\n=== Testing TStrongObjectPtr ==="));

	TStrongObjectPtr<USmartObject> StrongPtr(NewObject<USmartObject>());
	if (StrongPtr.IsValid())
	{
		StrongPtr->PrintInfo("TStrongObjectPtr");

		SimulateGarbageCollection();

		if (StrongPtr.IsValid())
		{
			StrongPtr->PrintInfo("TStrongObjectPtr (after GC)");
		}
		else
		{
			UE_LOG(LogTemp, Warning, TEXT("TStrongObjectPtr is invalid after GC!"));
		}
	}

	StrongPtr.Reset();
}

void USmartPointerTester::TestTWeakObjectPtr()
{
	UE_LOG(LogTemp, Log, TEXT("\n=== Testing TWeakObjectPtr ==="));

	USmartObject* RawObject = NewObject<USmartObject>();
	TWeakObjectPtr<USmartObject> WeakObjPtr(RawObject);

	if (WeakObjPtr.IsValid())
	{
		WeakObjPtr->PrintInfo("TWeakObjectPtr");

		RawObject->MarkAsGarbage();
		SimulateGarbageCollection();

		if (WeakObjPtr.IsValid())
		{
			WeakObjPtr->PrintInfo("TWeakObjectPtr (after GC)");
		}
		else
		{
			UE_LOG(LogTemp, Log, TEXT("TWeakObjectPtr is invalid after GC as expected"));
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

void USmartPointerTester::SimulateGarbageCollection()
{
	UE_LOG(LogTemp, Log, TEXT("Simulating garbage collection..."));
	CollectGarbage(GARBAGE_COLLECTION_KEEPFLAGS);
	FPlatformProcess::Sleep(0.1f);
}

IMPLEMENT_APPLICATION(SmartPointerTest, "SmartPointerTest");

INT32_MAIN_INT32_ARGC_TCHAR_ARGV()
{
	GEngineLoop.PreInit(ArgC, ArgV);
	UE_LOG(LogTemp, Log, TEXT("===== UE5 Smart Pointer Test Program Starting ====="));

	USmartPointerTester* Tester = NewObject<USmartPointerTester>(GetTransientPackage());
	Tester->TestAllSmartPointers();

	Tester->MarkAsGarbage();
	CollectGarbage(GARBAGE_COLLECTION_KEEPFLAGS);

	FEngineLoop::AppExit();
	return 0;
}

```
