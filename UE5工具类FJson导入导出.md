```c
// Copyright Epic Games, Inc. All Rights Reserved.


#include "BlankProgram.h"

#include "RequiredProgramMainCPPInclude.h"
#include "Serialization/JsonReader.h"
#include "Serialization/JsonSerializer.h"
#include "Serialization/JsonWriter.h"
#include "Misc/FileHelper.h"

DEFINE_LOG_CATEGORY_STATIC(LogBlankProgram, Log, All);
IMPLEMENT_APPLICATION(BlankProgram, "BlankProgram");

void SaveJsonToFile(const FString& FilePath, const TSharedPtr<FJsonObject>& JsonObject)
{
	// 将 JSON 对象转换为 JSON 字符串
	FString JsonString;
	TSharedRef<TJsonWriter<>> Writer = TJsonWriterFactory<>::Create(&JsonString);
	FJsonSerializer::Serialize(JsonObject.ToSharedRef(), Writer);

	// 将 JSON 字符串写入文件
	FFileHelper::SaveStringToFile(JsonString, *FilePath);
}

TSharedPtr<FJsonObject> LoadJsonFromFile(const FString& FilePath)
{
	// 从文件中读取 JSON 字符串
	FString JsonString;
	if (!FFileHelper::LoadFileToString(JsonString, *FilePath))
	{
		UE_LOG(LogBlankProgram, Display, TEXT("Failed to load file: %s"), *FilePath);
		return nullptr;
	}

	// 将 JSON 字符串解析为 JSON 对象
	TSharedPtr<FJsonObject> ParsedJsonObject;
	TSharedRef<TJsonReader<>> Reader = TJsonReaderFactory<>::Create(JsonString);
	if (!FJsonSerializer::Deserialize(Reader, ParsedJsonObject))
	{
		UE_LOG(LogBlankProgram, Display, TEXT("Failed to parse JSON string from file: %s"), *FilePath);
		return nullptr;
	}

	return ParsedJsonObject;
}

void TestJson()
{
	// 创建一个 JSON 对象
	TSharedPtr<FJsonObject> JsonObject = MakeShareable(new FJsonObject);
	JsonObject->SetStringField(TEXT("Name"), TEXT("John"));
	JsonObject->SetNumberField(TEXT("Age"), 30);

	// 保存 JSON 数据到文件
	FString FilePath = FPaths::ProjectContentDir() + TEXT("SaveData.json");
	SaveJsonToFile(FilePath, JsonObject);

	// 从文件中加载 JSON 数据
	TSharedPtr<FJsonObject> LoadedJsonObject = LoadJsonFromFile(FilePath);
	if (LoadedJsonObject.IsValid())
	{
		FString Name;
		int32 Age;
		if (LoadedJsonObject->TryGetStringField(TEXT("Name"), Name) &&
			LoadedJsonObject->TryGetNumberField(TEXT("Age"), Age))
		{
			UE_LOG(LogBlankProgram, Display, TEXT("Loaded Name: %s, Age: %d"), *Name, Age);
		}
	}
}

void TestComplexJson()
{
	// 创建一个复杂的 JSON 对象
	TSharedPtr<FJsonObject> JsonObject = MakeShareable(new FJsonObject);
	JsonObject->SetStringField(TEXT("Name"), TEXT("John"));
	JsonObject->SetNumberField(TEXT("Age"), 30);

	// 创建一个嵌套的 JSON 对象
	TSharedPtr<FJsonObject> AddressObject = MakeShareable(new FJsonObject);
	AddressObject->SetStringField(TEXT("Street"), TEXT("123 Main St"));
	AddressObject->SetStringField(TEXT("City"), TEXT("Anytown"));
	AddressObject->SetStringField(TEXT("State"), TEXT("CA"));
	AddressObject->SetStringField(TEXT("ZipCode"), TEXT("12345"));
	JsonObject->SetObjectField(TEXT("Address"), AddressObject);

	// 创建一个 JSON 数组
	TArray<TSharedPtr<FJsonValue>> ScoresArray;
	ScoresArray.Add(MakeShareable(new FJsonValueNumber(90)));
	ScoresArray.Add(MakeShareable(new FJsonValueNumber(85)));
	ScoresArray.Add(MakeShareable(new FJsonValueNumber(95)));
	ScoresArray.Add(MakeShareable(new FJsonValueNumber(88)));
	JsonObject->SetArrayField(TEXT("Scores"), ScoresArray);

	// 保存 JSON 数据到文件
	FString FilePath = FPaths::ProjectContentDir() + TEXT("ComplexData.json");
	SaveJsonToFile(FilePath, JsonObject);

	// 从文件中加载 JSON 数据
	TSharedPtr<FJsonObject> LoadedJsonObject = LoadJsonFromFile(FilePath);
	if (LoadedJsonObject.IsValid())
	{
		FString Name;
		int32 Age;
		if (LoadedJsonObject->TryGetStringField(TEXT("Name"), Name) &&
			LoadedJsonObject->TryGetNumberField(TEXT("Age"), Age))
		{
			UE_LOG(LogBlankProgram, Display, TEXT("Loaded Name: %s, Age: %d"), *Name, Age);

			// 获取嵌套的 JSON 对象
			TSharedPtr<FJsonObject> LoadedAddressObject = LoadedJsonObject->GetObjectField(TEXT("Address"));
			if (LoadedAddressObject.IsValid())
			{
				FString Street, City, State, ZipCode;
				if (LoadedAddressObject->TryGetStringField(TEXT("Street"), Street) &&
					LoadedAddressObject->TryGetStringField(TEXT("City"), City) &&
					LoadedAddressObject->TryGetStringField(TEXT("State"), State) &&
					LoadedAddressObject->TryGetStringField(TEXT("ZipCode"), ZipCode))
				{
					UE_LOG(LogBlankProgram, Display, TEXT("Loaded Address: %s, %s, %s, %s"), *Street, *City, *State, *ZipCode);
				}
			}

			// 获取 JSON 数组
			TArray<TSharedPtr<FJsonValue>> LoadedScores = LoadedJsonObject->GetArrayField(TEXT("Scores"));
			for (int32 i = 0; i < LoadedScores.Num(); ++i)
			{
				int32 Score;
				if (LoadedScores[i]->TryGetNumber(Score))
				{
					UE_LOG(LogBlankProgram, Display, TEXT("Loaded Score %d: %d"), i, Score);
				}
			}
		}
	}
}

struct HouseFurnitureInfo
{
	int32 HouseId = -1;
	int32 InstanceId = -1;
	int32 TmplId = -1;
	int32 RelyId = -1;
	FVector Location = FVector::ZeroVector;
	FRotator Rotation = FRotator::ZeroRotator;
};

void TestPlayerHomeJson()
{
	TSharedPtr<FJsonObject> JsonObject = MakeShared<FJsonObject>();
	JsonObject->SetNumberField(TEXT("HouseId"), 36);
	TSharedPtr<FJsonObject> InstancesObject = MakeShared<FJsonObject>();
	for (int Idx = 0; Idx < 2; Idx++)
	{
		TSharedPtr<FJsonObject> InstanceObject = MakeShared<FJsonObject>();
		InstanceObject->SetNumberField(TEXT("HouseId"), 36);
		InstanceObject->SetNumberField(TEXT("InstanceId"), Idx);
		InstanceObject->SetNumberField(TEXT("TmplId"), 14828);
		InstanceObject->SetNumberField(TEXT("RelyId"), 5);
		TArray<TSharedPtr<FJsonValue>> Location;
		Location.Add(MakeShared<FJsonValueNumber>(Idx + 10));
		Location.Add(MakeShared<FJsonValueNumber>(Idx + 20));
		Location.Add(MakeShared<FJsonValueNumber>(Idx + 30));
		InstanceObject->SetArrayField(TEXT("Location"), Location);
		TArray<TSharedPtr<FJsonValue>> Rotation;
		Rotation.Add(MakeShared<FJsonValueNumber>(0));
		Rotation.Add(MakeShared<FJsonValueNumber>(Idx + 45));
		Rotation.Add(MakeShared<FJsonValueNumber>(0));
		InstanceObject->SetArrayField(TEXT("Rotation"), Rotation);
		InstancesObject->SetObjectField(FString::Printf(TEXT("%d"), Idx), InstanceObject);
	}
	JsonObject->SetObjectField(TEXT("Instances"), InstancesObject);

	FString FilePath = FPaths::ProjectContentDir() + TEXT("PlayerHome.json");
	SaveJsonToFile(FilePath, JsonObject);

    //////////////////////////////////////////////////////////////////////////////////////////////

	HouseFurnitureInfo LoadInfo;
	FString LoadHouseInfoLog;
	TSharedPtr<FJsonObject> LoadedJsonObject = LoadJsonFromFile(FilePath);
	if (LoadedJsonObject.IsValid())
	{
		int LoadHouseId;
		bool bRet = LoadedJsonObject->TryGetNumberField(TEXT("HouseId"), LoadHouseId);
		UE_LOG(LogBlankProgram, Display, TEXT("Loaded HouseId: %d"), LoadHouseId);
		TSharedPtr<FJsonObject> InstancesObjects = LoadedJsonObject->GetObjectField(TEXT("Instances"));
		for (int Idx = 0; Idx < InstancesObjects->Values.Num(); Idx++)
		{
			TSharedPtr<FJsonObject> InstanceObject = InstancesObjects->GetObjectField(FString::Printf(TEXT("%d"), Idx));
			if (InstanceObject.IsValid())
			{
				int32 HouseId, InstanceId, TmplId, RelyId;
				bRet &= InstanceObject->TryGetNumberField(TEXT("HouseId"), HouseId);
				bRet &= InstanceObject->TryGetNumberField(TEXT("InstanceId"), InstanceId);
				bRet &= InstanceObject->TryGetNumberField(TEXT("TmplId"), TmplId);
				bRet &= InstanceObject->TryGetNumberField(TEXT("RelyId"), RelyId);
				UE_LOG(LogBlankProgram, Display, TEXT("Loaded Instance[%d] HouseId:%d, InstanceId:%d, TmplId:%d, RelyId:%d"), Idx, HouseId, InstanceId, TmplId, RelyId);
				const TArray<TSharedPtr<FJsonValue>>* Location;
				TArray<int32> LocationArray;
				bRet &= InstanceObject->TryGetArrayField(TEXT("Location"), Location);
				for (TSharedPtr<FJsonValue> Val : *Location)
				{
					LocationArray.Add(Val->AsNumber());
					UE_LOG(LogBlankProgram, Display, TEXT("Loaded Instance[%d] Loction:%f"), Idx, Val->AsNumber());
				}
				const TArray<TSharedPtr<FJsonValue>>* Rotation;
				TArray<int32> RotationArray;
				bRet &= InstanceObject->TryGetArrayField(TEXT("Rotation"), Rotation);
				for (TSharedPtr<FJsonValue> Val : *Rotation)
				{
					RotationArray.Add(Val->AsNumber());
					UE_LOG(LogBlankProgram, Display, TEXT("Loaded Instance[%d] Rotation:%f"), Idx, Val->AsNumber());
				}
			}
		}
	}
}

INT32_MAIN_INT32_ARGC_TCHAR_ARGV()
{
	GEngineLoop.PreInit(ArgC, ArgV);
	UE_LOG(LogBlankProgram, Display, TEXT("Hello World"));
	//TestJson();
	//TestComplexJson();
	TestPlayerHomeJson();
	while (true)
	{

	}
	FEngineLoop::AppExit();
	return 0;
}

```

