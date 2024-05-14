## 使用FJson

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

## 使用JsonCpp

```cpp
// Copyright Epic Games, Inc. All Rights Reserved.


#include "BlankProgram.h"
#include "RequiredProgramMainCPPInclude.h"

#ifndef TEMPLATELIB_API
#define TEMPLATELIB_API
#endif

#include <iostream>
#include <fstream>
#include <array>
#include <memory>
#include <unordered_map>
#include "json.h"

DEFINE_LOG_CATEGORY_STATIC(LogBlankProgram, Log, All);
IMPLEMENT_APPLICATION(BlankProgram, "BlankProgram");

struct HouseFurnitureInfo
{
	HouseFurnitureInfo(int HouseId, int InstanceId, int TmplId, int  RelyId, std::array<double, 3> Location, std::array<double, 3> Rotation) : HouseId(HouseId), InstanceId(InstanceId), TmplId(TmplId), RelyId(RelyId), Location(Location), Rotation(Rotation) {}
	int HouseId = -1;
	int InstanceId = -1;
	int TmplId = -1;
	int RelyId = -1;
	std::array<double, 3> Location;
	std::array<double, 3> Rotation;
};

class PlayerHomeFunitureJsonData
{
public:
	static void SaveJsonToFile(const std::string& FilePath, const Json::Value& JsonObject)
	{
		std::ofstream File(FilePath);
		if (!File.is_open())
		{
			std::cerr << "Failed to open file: " << FilePath << std::endl;
			return;
		}

		Json::StreamWriterBuilder Builder;
		Builder["indentation"] = "\t";
		std::unique_ptr<Json::StreamWriter> Writer(Builder.newStreamWriter());
		Writer->write(JsonObject, &File);
	}

	static Json::Value LoadJsonFromFile(const std::string& FilePath)
	{
		std::ifstream File(FilePath);
		if (!File.is_open())
		{
			std::cerr << "Failed to open file: " << FilePath << std::endl;
			return Json::Value();
		}

		Json::Value JsonObject;
		Json::CharReaderBuilder Builder;
		std::string Error;
		if (!Json::parseFromStream(Builder, File, &JsonObject, &Error))
		{
			std::cerr << "Failed to parse JSON string from file: " << FilePath << std::endl;
			return Json::Value();
		}

		return JsonObject;
	}

	static void SaveMonoHouseFurnitureJson(int HouseId, std::vector<HouseFurnitureInfo*> HouseInfo)
	{
		Json::Value JsonObject;
		Json::Value InstancesObject(Json::objectValue);

		int FurnitureIdx = 0;
		for (const auto& FurnitureInfo : HouseInfo)
		{
			Json::Value InstanceObject(Json::objectValue);
			InstanceObject["HouseId"] = FurnitureInfo->HouseId;
			InstanceObject["InstanceId"] = FurnitureInfo->InstanceId;
			InstanceObject["TmplId"] = FurnitureInfo->TmplId;
			InstanceObject["RelyId"] = FurnitureInfo->RelyId;

			Json::Value Location(Json::arrayValue);
			Location.append(FurnitureInfo->Location[0]);
			Location.append(FurnitureInfo->Location[1]);
			Location.append(FurnitureInfo->Location[2]);
			InstanceObject["Location"] = Location;

			Json::Value Rotation(Json::arrayValue);
			Rotation.append(FurnitureInfo->Rotation[0]);
			Rotation.append(FurnitureInfo->Rotation[1]);
			Rotation.append(FurnitureInfo->Rotation[2]);
			InstanceObject["Rotation"] = Rotation;
			InstancesObject[std::to_string(FurnitureIdx)] = InstanceObject;
			FurnitureIdx++;
		}
		JsonObject[std::to_string(0)] = InstancesObject;

		std::string FilePath = "HouseId_" + std::to_string(HouseId) + "_FunitureData.json";
		SaveJsonToFile(FilePath, JsonObject);
	}

	static void SaveAllHousesFurnitureJson(std::vector<std::vector<HouseFurnitureInfo*>> AllHousesInfo)
	{
		Json::Value JsonObject;
		int HouseIdx = 0;
		for (const auto& HouseInfo : AllHousesInfo)
		{
			Json::Value InstancesObject(Json::objectValue);
			int FurnitureIdx = 0;
			for (const auto& FurnitureInfo : HouseInfo)
			{
				Json::Value InstanceObject(Json::objectValue);
				InstanceObject["HouseId"] = FurnitureInfo->HouseId;
				InstanceObject["InstanceId"] = FurnitureInfo->InstanceId;
				InstanceObject["TmplId"] = FurnitureInfo->TmplId;
				InstanceObject["RelyId"] = FurnitureInfo->RelyId;

				Json::Value Location(Json::arrayValue);
				Location.append(FurnitureInfo->Location[0]);
				Location.append(FurnitureInfo->Location[1]);
				Location.append(FurnitureInfo->Location[2]);
				InstanceObject["Location"] = Location;

				Json::Value Rotation(Json::arrayValue);
				Rotation.append(FurnitureInfo->Rotation[0]);
				Rotation.append(FurnitureInfo->Rotation[1]);
				Rotation.append(FurnitureInfo->Rotation[2]);
				InstanceObject["Rotation"] = Rotation;
				InstancesObject[std::to_string(FurnitureIdx)] = InstanceObject;
				FurnitureIdx++;
			}
			JsonObject[std::to_string(HouseIdx)] = InstancesObject;
			HouseIdx++;
		}

		std::string FilePath = "AllHouses_FunitureData.json";
		SaveJsonToFile(FilePath, JsonObject);
	}

	static void SaveFurnitureDataInternal(Json::Value& JsonObject, Json::Value& InstancesObject, std::vector<HouseFurnitureInfo*> HouseInfo)
	{
		int FurnitureIdx = 0;
		for (const auto& FurnitureInfo : HouseInfo)
		{
			Json::Value InstanceObject(Json::objectValue);
			InstanceObject["HouseId"] = FurnitureInfo->HouseId;
			InstanceObject["InstanceId"] = FurnitureInfo->InstanceId;
			InstanceObject["TmplId"] = FurnitureInfo->TmplId;
			InstanceObject["RelyId"] = FurnitureInfo->RelyId;

			Json::Value Location(Json::arrayValue);
			Location.append(FurnitureInfo->Location[0]);
			Location.append(FurnitureInfo->Location[1]);
			Location.append(FurnitureInfo->Location[2]);
			InstanceObject["Location"] = Location;

			Json::Value Rotation(Json::arrayValue);
			Rotation.append(FurnitureInfo->Rotation[0]);
			Rotation.append(FurnitureInfo->Rotation[1]);
			Rotation.append(FurnitureInfo->Rotation[2]);
			InstanceObject["Rotation"] = Rotation;
			InstancesObject[std::to_string(FurnitureIdx)] = InstanceObject;
			FurnitureIdx++;
		}
	}

	static void MonoSavePlayerHomeJson(int HouseId, std::vector<HouseFurnitureInfo*> HouseInfo)
	{
		Json::Value JsonObject;
		Json::Value InstancesObject(Json::objectValue);
		SaveFurnitureDataInternal(JsonObject, InstancesObject, HouseInfo);
		JsonObject[std::to_string(0)] = InstancesObject;
		std::string FilePath = "HouseId_" + std::to_string(HouseId) + "_FunitureData.json";
		SaveJsonToFile(FilePath, JsonObject);
	}

	static void MultiSavePlayerHomeJson(std::vector<std::vector<HouseFurnitureInfo*>> AllHousesInfo)
	{
		Json::Value JsonObject;
		int HouseIdx = 0;
		for (const auto& HouseInfo : AllHousesInfo)
		{
			Json::Value InstancesObject(Json::objectValue);
			SaveFurnitureDataInternal(JsonObject, InstancesObject, HouseInfo);
			JsonObject[std::to_string(HouseIdx)] = InstancesObject;
			HouseIdx++;
		}
		std::string FilePath = "AllHouses_FunitureData.json";
		SaveJsonToFile(FilePath, JsonObject);
	}

	static void LoadMonoHouseFurnitureJson(int HouseId, std::vector<std::shared_ptr<HouseFurnitureInfo>>& HouseInfo)
	{
		std::string FilePath = "HouseId_" + std::to_string(HouseId) + "_FunitureData.json";
		Json::Value LoadedJsonObject = LoadJsonFromFile(FilePath);
		if (LoadedJsonObject.empty()) return;
		Json::Value InstancesObjects = LoadedJsonObject["0"];
		if (InstancesObjects.empty()) return;
		for (unsigned int Idx = 0; Idx < InstancesObjects.size(); Idx++)
		{
			Json::Value InstanceObject = InstancesObjects[std::to_string(Idx)];
			if (InstanceObject.empty()) return;
			int HouseId = InstanceObject["HouseId"].asInt();
			int InstanceId = InstanceObject["InstanceId"].asInt();
			int TmplId = InstanceObject["TmplId"].asInt();
			int RelyId = InstanceObject["RelyId"].asInt();
			std::cout << "Loaded Instance HouseId:" << HouseId << ", InstanceId:" << InstanceId << ", TmplId:" << TmplId << ", RelyId:" << RelyId << std::endl;

			std::array<double, 3> LocationArray;
			const Json::Value Location = InstanceObject["Location"];
			for (unsigned int i = 0; i < Location.size(); i++)
			{
				LocationArray[i] = Location[i].asDouble();
				std::cout << "Loaded Instance Location:" << LocationArray[i] << std::endl;
			}

			std::array<double, 3> RotationArray;
			const Json::Value Rotation = InstanceObject["Rotation"];
			for (unsigned int i = 0; i < Rotation.size(); i++)
			{
				RotationArray[i] = Rotation[i].asDouble();
				std::cout << "Loaded Instance Rotation:" << RotationArray[i] << std::endl;
			}

			std::shared_ptr<HouseFurnitureInfo> FurnitureInfo = std::make_shared< HouseFurnitureInfo>(HouseId, InstanceId, TmplId, RelyId, LocationArray, RotationArray);
			HouseInfo.push_back(FurnitureInfo);
		}
	}

	static void LoadAllHousesFurnitureJson(std::vector<std::vector<std::shared_ptr<HouseFurnitureInfo>>>& AllHousesInfo)
	{
		std::string FilePath = "AllHouses_FunitureData.json";
		Json::Value LoadedJsonObject = LoadJsonFromFile(FilePath);
		if (LoadedJsonObject.empty()) return;
		for (unsigned int HouseIdx = 0; HouseIdx < LoadedJsonObject.size(); HouseIdx++)
		{
			Json::Value InstancesObjects = LoadedJsonObject[std::to_string(HouseIdx)];
			if (InstancesObjects.empty()) return;
			std::vector<std::shared_ptr<HouseFurnitureInfo>> HouseInfo;
			for (unsigned int FunitureIdx = 0; FunitureIdx < InstancesObjects.size(); FunitureIdx++)
			{
				Json::Value InstanceObject = InstancesObjects[std::to_string(FunitureIdx)];
				if (InstanceObject.empty()) return;
				int HouseId = InstanceObject["HouseId"].asInt();
				int InstanceId = InstanceObject["InstanceId"].asInt();
				int TmplId = InstanceObject["TmplId"].asInt();
				int RelyId = InstanceObject["RelyId"].asInt();
				std::cout << "Loaded Instance HouseId:" << HouseId << ", InstanceId:" << InstanceId << ", TmplId:" << TmplId << ", RelyId:" << RelyId << std::endl;

				std::array<double, 3> LocationArray;
				const Json::Value Location = InstanceObject["Location"];
				for (unsigned int i = 0; i < Location.size(); i++)
				{
					LocationArray[i] = Location[i].asDouble();
					std::cout << "Loaded Instance Location:" << LocationArray[i] << std::endl;
				}

				std::array<double, 3> RotationArray;
				const Json::Value Rotation = InstanceObject["Rotation"];
				for (unsigned int i = 0; i < Rotation.size(); i++)
				{
					RotationArray[i] = Rotation[i].asDouble();
					std::cout << "Loaded Instance Rotation:" << RotationArray[i] << std::endl;
				}

				std::shared_ptr<HouseFurnitureInfo> FurnitureInfo = std::make_shared< HouseFurnitureInfo>(HouseId, InstanceId, TmplId, RelyId, LocationArray, RotationArray);
				HouseInfo.push_back(FurnitureInfo);
			}
			AllHousesInfo.push_back(HouseInfo);
		}
	}

	static void LoadFurnitureDataInternal(Json::Value& InstancesObjects, std::vector<std::shared_ptr<HouseFurnitureInfo>>& HouseInfo)
	{
		for (unsigned int Idx = 0; Idx < InstancesObjects.size(); Idx++)
		{
			Json::Value InstanceObject = InstancesObjects[std::to_string(Idx)];
			if (InstanceObject.empty()) return;
			int HouseId = InstanceObject["HouseId"].asInt();
			int InstanceId = InstanceObject["InstanceId"].asInt();
			int TmplId = InstanceObject["TmplId"].asInt();
			int RelyId = InstanceObject["RelyId"].asInt();
			std::cout << "Loaded Instance HouseId:" << HouseId << ", InstanceId:" << InstanceId << ", TmplId:" << TmplId << ", RelyId:" << RelyId << std::endl;

			std::array<double, 3> LocationArray;
			const Json::Value Location = InstanceObject["Location"];
			for (unsigned int i = 0; i < Location.size(); i++)
			{
				LocationArray[i] = Location[i].asDouble();
				std::cout << "Loaded Instance Location:" << LocationArray[i] << std::endl;
			}

			std::array<double, 3> RotationArray;
			const Json::Value Rotation = InstanceObject["Rotation"];
			for (unsigned int i = 0; i < Rotation.size(); i++)
			{
				RotationArray[i] = Rotation[i].asDouble();
				std::cout << "Loaded Instance Rotation:" << RotationArray[i] << std::endl;
			}

			std::shared_ptr<HouseFurnitureInfo> FurnitureInfo = std::make_shared< HouseFurnitureInfo>(HouseId, InstanceId, TmplId, RelyId, LocationArray, RotationArray);
			HouseInfo.push_back(FurnitureInfo);
		}
	}

	static void MonoLoadPlayerHomeJson(int HouseId, std::vector<std::shared_ptr<HouseFurnitureInfo>>& HouseInfo)
	{
		std::string FilePath = "HouseId_" + std::to_string(HouseId) + "_FunitureData.json";
		Json::Value LoadedJsonObject = LoadJsonFromFile(FilePath);
		if (LoadedJsonObject.empty()) return;
		Json::Value InstancesObjects = LoadedJsonObject["0"];
		if (InstancesObjects.empty()) return;
		LoadFurnitureDataInternal(InstancesObjects, HouseInfo);
	}

	static void MultiLoadPlayerHomeJson(std::vector<std::vector<std::shared_ptr<HouseFurnitureInfo>>>& AllHousesInfo)
	{
		std::string FilePath = "AllHouses_FunitureData.json";
		Json::Value LoadedJsonObject = LoadJsonFromFile(FilePath);
		if (LoadedJsonObject.empty()) return;
		for (unsigned int HouseIdx = 0; HouseIdx < LoadedJsonObject.size(); HouseIdx++)
		{
			Json::Value InstancesObjects = LoadedJsonObject[std::to_string(HouseIdx)];
			if (InstancesObjects.empty()) return;
			std::vector<std::shared_ptr<HouseFurnitureInfo>> HouseInfo;
			LoadFurnitureDataInternal(InstancesObjects, HouseInfo);
			AllHousesInfo.push_back(HouseInfo);
		}
	}
};

INT32_MAIN_INT32_ARGC_TCHAR_ARGV()
{
	GEngineLoop.PreInit(ArgC, ArgV);
	UE_LOG(LogBlankProgram, Display, TEXT("Hello World"));
    {
		std::unordered_map<int, std::unordered_map<int, std::shared_ptr<HouseFurnitureInfo>>> AllFurnitureBases;
        std::unordered_map<int, std::shared_ptr<HouseFurnitureInfo>> HouseInfo100;
		HouseInfo100[1] = std::make_shared<HouseFurnitureInfo>(100, 1, 3, 4, std::array<double, 3>{ 5.1, 6.1, 7.1 }, std::array<double, 3>{ 8.2, 9.2, 10.2 });
		HouseInfo100[2] = std::make_shared<HouseFurnitureInfo>(100, 2, 3, 4, std::array<double, 3>{ 5.1, 6.1, 7.1 }, std::array<double, 3>{ 8.2, 9.2, 10.2 });
		AllFurnitureBases[100] = HouseInfo100;
        std::unordered_map<int, std::shared_ptr<HouseFurnitureInfo>> HouseInfo101;
		HouseInfo101[1] = std::make_shared<HouseFurnitureInfo>(101, 1, 3, 4, std::array<double, 3>{ 5.1, 6.1, 7.1 }, std::array<double, 3>{ 8.2, 9.2, 10.2 });
		HouseInfo101[2] = std::make_shared<HouseFurnitureInfo>(101, 2, 3, 4, std::array<double, 3>{ 5.1, 6.1, 7.1 }, std::array<double, 3>{ 8.2, 9.2, 10.2 });
		AllFurnitureBases[101] = HouseInfo101;
        /////////////////////////////////////////////////////////////////////
        {
			std::vector<HouseFurnitureInfo*> HouseInfo;
			HouseInfo.push_back(HouseInfo100[1].get());
			HouseInfo.push_back(HouseInfo100[2].get());
			PlayerHomeFunitureJsonData::MonoSavePlayerHomeJson(100, HouseInfo);
            ///////////////////////////////////////////////////////////
			std::vector<std::shared_ptr<HouseFurnitureInfo>> LoadHouseInfo;
			PlayerHomeFunitureJsonData::MonoLoadPlayerHomeJson(100, LoadHouseInfo);
        }
		std::cout << "/////////////////////////////////////////////////////" << std::endl;
        {
            std::vector<std::vector<HouseFurnitureInfo*>> AllHouseInfo;
			std::vector<HouseFurnitureInfo*> HouseInfo;
            HouseInfo.push_back(HouseInfo100[1].get());
            HouseInfo.push_back(HouseInfo100[2].get());
			std::vector<HouseFurnitureInfo*> HouseInfo2;
            HouseInfo2.push_back(HouseInfo101[1].get());
            HouseInfo2.push_back(HouseInfo101[2].get());
            AllHouseInfo.push_back(HouseInfo);
            AllHouseInfo.push_back(HouseInfo2);
			PlayerHomeFunitureJsonData::MultiSavePlayerHomeJson(AllHouseInfo);
            ///////////////////////////////////////////////////////////
			std::vector<std::vector<std::shared_ptr<HouseFurnitureInfo>>> AllLoadHousesInfo;
			PlayerHomeFunitureJsonData::MultiLoadPlayerHomeJson(AllLoadHousesInfo);
        }
    }
	while (true) {};
	FEngineLoop::AppExit();
	return 0;
}

```

