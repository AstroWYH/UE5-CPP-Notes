# Build Configuration Descriptions

所有*编辑器目标都包括游戏代码和编辑器，当您运行它们时，它们将打开编辑器。 Debug / DebugGame 目标不包括编辑器，当您运行它们时，它将启动游戏。

### 构建配置描述

Unreal Engine 4 uses a custom building method via the UnrealBuildTool (UBT). This tool processes the information necessary to build the engine's reflection system, integrating your C++ code with Blueprints, replication, serialization, and garbage collection.
虚幻引擎 4 通过 UnrealBuildTool (UBT) 使用自定义构建方法。该工具处理构建引擎反射系统所需的信息，将 C++ 代码与蓝图、复制、序列化和垃圾收集集成。

Every build configuration contains two keywords, and the first keyword indicates the state of the engine and your game project. For instance, if you compile using a Debug configuration, you will be able to debug your game's code. The second keyword indicates the target you are building for. For example, if you want to open a project in Unreal, you need to build with the Editor target keyword.
每个构建配置都包含两个关键字，第一个关键字表示引擎和游戏项目的状态。例如，如果您使用调试配置进行编译，您将能够调试游戏的代码。第二个关键字表示您正在构建的目标。例如，如果您想在 Unreal 中打开一个项目，则需要使用 Editor target 关键字进行构建。

### Debug 调试

This configuration contains symbols for debugging. This configuration builds both engine and game code in debug configuration. If you compile your project using the Debug configuration and want to open the project with the Unreal Editor, you must use the "-debug" flag in order to see your code changes reflected in your project.
此配置包含用于调试的符号。此配置在调试配置中构建引擎和游戏代码。如果您使用调试配置编译项目并希望使用虚幻编辑器打开项目，则必须使用“-debug”标志才能查看项目中反映的代码更改。

### DebugGame 调试游戏

This configuration builds the engine as optimized, but leaves the game code debuggable. This configuration is ideal for debugging only game modules.
此配置构建优化的引擎，但使游戏代码可调试。此配置非常适合仅调试游戏模块。

### Development 

This configuration enables all but the most time-consuming engine and game code optimizations, which makes it ideal for development and performance reasons. Unreal Editor uses the Development configuration by default. Compiling your project using the Development configuration enables you to see code changes made to your project reflected in the editor.
此配置可以实现除最耗时的引擎和游戏代码之外的所有优化，这使其成为开发和性能方面的理想选择。虚幻编辑器默认使用开发配置。使用开发配置编译项目使您能够看到编辑器中反映的对项目所做的代码更改。

### Shipping 

This is the configuration for optimal performance and shipping your game. This configuration strips out console commands, stats, and profiling tools.
这是实现最佳性能和交付游戏的配置。此配置去掉了控制台命令、统计数据和分析工具。

### Test 测试

This configuration is the Shipping configuration, but with some console commands, stats, and profiling tools enabled.
此配置是运输配置，但启用了一些控制台命令、统计数据和分析工具。

----------------------------------------------

### [empty] [空的]

This configuration builds a stand-alone executable version of your project, but requires cooked content specific to the platform. Please refer to our Packaging Projects Reference page to learn more about cooked content.
此配置构建项目的独立可执行版本，但需要特定于平台的熟化内容。请参阅我们的包装项目参考页面，了解有关熟化内容的更多信息。

### Editor 编辑

To be able to open a project in Unreal Editor and see all code changes reflected, the project must be built in an Editor configuration.
为了能够在虚幻编辑器中打开项目并查看反映的所有代码更改，必须在编辑器配置中构建该项目。

### Client 客户端

If you're working on a multiplayer project using UE4 networking features, this target designates the specified project as being a Client in UE4's client-server model for multiplayer games. If there is a <Game>Client.Target.cs file, the Client build configurations will be valid.
如果您正在使用 UE4 网络功能开发多人游戏项目，则此目标会将指定项目指定为 UE4 多人游戏客户端-服务器模型中的客户端。如果存在 <Game>Client.Target.cs 文件，则客户端构建配置将有效。

### Server 服务器

If you're working on a multiplayer project using UE4 networking features, this target designates the specified project as being a Server in UE4's client-server model for multiplayer games. If there is a <Game>Server.Target.cs file, the Server build configurations will be valid.
如果您正在使用 UE4 网络功能开发多人游戏项目，则此目标会将指定项目指定为 UE4 多人游戏客户端-服务器模型中的服务器。如果存在 <Game>Server.Target.cs 文件，则服务器构建配置将有效。
