实例举例：

FArchive* Reader = IFileManager::Get().CreateFileReader(*Filename, 0);

这样，Reader这个FArchive的Serialize序列化，都用于读取。

比如以下操作：

Reader->Seek(SomeOffset); // 指针移动到目的位置

Reader->Serialize(&SomeStruct, sizeof(SomeStruct)); // 读取SomeStruct



FArchive 是 Unreal Engine 中用于序列化（将数据转换为可存储或传输的格式）和反序列化（从存储或传输的格式还原数据）的基类。在 Unreal Engine 中，它主要用于将数据存储到文件或网络中，以及从这些来源读取数据。

```cpp
#include "Misc/FileHelper.h"
#include "HAL/PlatformFilemanager.h"

// 定义一个简单的数据结构
struct FMyData
{
    FString Name;
    int32 Score;
};

// 将数据保存到文件
bool SaveDataToFile(const FString& FilePath, const FMyData& Data)
{
    // 创建一个二进制存档
    FBufferArchive BufferArchive;

    // 将数据序列化到存档中
    BufferArchive << Data.Name;
    BufferArchive << Data.Score;

    // 将存档的内容保存到文件
    return FFileHelper::SaveArrayToFile(BufferArchive.GetData(), *FilePath);
}

// 从文件加载数据
bool LoadDataFromFile(const FString& FilePath, FMyData& OutData)
{
    // 读取文件内容到数组
    TArray<uint8> FileData;
    if (!FFileHelper::LoadFileToArray(FileData, *FilePath))
    {
        return false;
    }

    // 创建一个二进制存档并将文件内容反序列化到数据结构中
    FMemoryReader MemoryReader(FileData, true);
    MemoryReader << OutData.Name;
    MemoryReader << OutData.Score;

    return true;
}

// 在某个地方调用 SaveDataToFile 和 LoadDataFromFile 来保存和加载数据

#include "Misc/FileHelper.h"
#include "HAL/PlatformFilemanager.h"

// 定义一个简单的数据结构
struct FMyData
{
    FString Name;
    int32 Score;

    // 自定义序列化方法
    void Serialize(FArchive& Ar)
    {
        Ar << Name;
        Ar << Score;
    }
};

// 将数据保存到文件
bool SaveDataToFile(const FString& FilePath, const FMyData& Data)
{
    // 创建一个二进制存档
    FBufferArchive BufferArchive;

    // 调用自定义序列化方法将数据序列化到存档中
    Data.Serialize(BufferArchive);

    // 将存档的内容保存到文件
    return FFileHelper::SaveArrayToFile(BufferArchive.GetData(), *FilePath);
}

// 从文件加载数据
bool LoadDataFromFile(const FString& FilePath, FMyData& OutData)
{
    // 读取文件内容到数组
    TArray<uint8> FileData;
    if (!FFileHelper::LoadFileToArray(FileData, *FilePath))
    {
        return false;
    }

    // 创建一个二进制存档并将文件内容反序列化到数据结构中
    FMemoryReader MemoryReader(FileData, true);
    
    // 调用自定义序列化方法将数据反序列化到数据结构中
    OutData.Serialize(MemoryReader);

    return true;
}

// 在某个地方调用 SaveDataToFile 和 LoadDataFromFile 来保存和加载数据
```
