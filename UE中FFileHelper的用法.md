FFileHelper 是 Unreal Engine 中的一个工具类，用于简化文件读写的操作。它提供了一些静态方法，方便在蓝图和 C++ 代码中进行文件的读写操作。以下是一些常用的 FFileHelper 方法的示例：

## 读取文本文件内容
```cpp
#include "Misc/FileHelper.h"

FString FileContents;
FString FilePath = TEXT("Path/To/Your/File.txt");

if (FFileHelper::LoadFileToString(FileContents, *FilePath))
{
    // 文件内容加载成功
    UE_LOG(LogTemp, Warning, TEXT("File Contents: %s"), *FileContents);
}
else
{
    // 文件加载失败
    UE_LOG(LogTemp, Error, TEXT("Failed to load file: %s"), *FilePath);
}
```

## 写入文本到文件
```cpp
#include "Misc/FileHelper.h"

FString ContentsToWrite = TEXT("Hello, World!");
FString FilePath = TEXT("Path/To/Your/File.txt");

if (FFileHelper::SaveStringToFile(ContentsToWrite, *FilePath))
{
    // 写入文件成功
    UE_LOG(LogTemp, Warning, TEXT("File written successfully: %s"), *FilePath);
}
else
{
    // 写入文件失败
    UE_LOG(LogTemp, Error, TEXT("Failed to write to file: %s"), *FilePath);
}
```

## 读取二进制文件内容
```cpp
#include "Misc/FileHelper.h"

TArray<uint8> FileContents;
FString FilePath = TEXT("Path/To/Your/BinaryFile.bin");

if (FFileHelper::LoadFileToArray(FileContents, *FilePath))
{
    // 二进制文件内容加载成功
    // FileContents 数组中包含文件的字节数据
    UE_LOG(LogTemp, Warning, TEXT("Binary file loaded successfully: %s"), *FilePath);
}
else
{
    // 文件加载失败
    UE_LOG(LogTemp, Error, TEXT("Failed to load binary file: %s"), *FilePath);
}

```
## 写入二进制数据到文件
```cpp
#include "Misc/FileHelper.h"

TArray<uint8> DataToWrite;
// 假设 DataToWrite 数组已经包含了你要写入的二进制数据

FString FilePath = TEXT("Path/To/Your/BinaryFile.bin");

if (FFileHelper::SaveArrayToFile(DataToWrite, *FilePath))
{
    // 写入文件成功
    UE_LOG(LogTemp, Warning, TEXT("Binary file written successfully: %s"), *FilePath);
}
else
{
    // 写入文件失败
    UE_LOG(LogTemp, Error, TEXT("Failed to write to binary file: %s"), *FilePath);
}
```
