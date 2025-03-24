以下是 `clangd` 全部功能的翻译和解释，基于你提供的帮助信息：

---

### **1. 通用选项 (Generic Options)**
- `--help`  
  显示可用的选项（使用 `--help-hidden` 显示更多选项）。  
- `--help-list`  
  显示可用选项的列表（使用 `--help-list-hidden` 显示更多选项）。  
- `--version`  
  显示程序的版本信息。

---

### **2. 编译标志选项 (clangd Compilation Flags Options)**
- `--compile-commands-dir=<string>`  
  指定查找 `compile_commands.json` 的路径。如果路径无效，`clangd` 会在当前目录和每个源文件的父目录中查找。  
- `--query-driver=<string>`  
  用于白名单的 GCC 兼容驱动程序的逗号分隔列表。匹配这些模式的驱动程序将用于提取系统包含路径。例如：`/usr/bin/**/clang-*,/path/to/repo/**/g++-*`。

---

### **3. 功能选项 (clangd Feature Options)**
- `--all-scopes-completion`  
  如果设置为 `true`，代码补全将包括不在当前作用域（如命名空间）中定义的符号。这些补全可能会插入作用域限定符。  
- `--background-index`  
  在后台索引项目代码，并将索引持久化到磁盘。  
- `--background-index-priority=<value>`  
  后台索引构建的线程优先级。此标志的效果取决于操作系统。  
  - `background`：最低优先级，在空闲 CPU 上运行。可能会留下“性能”核心未使用。  
  - `low`：比交互式工作优先级低。  
  - `normal`：与 `clangd` 的其他工作优先级相同。  
- `--clang-tidy`  
  启用 `clang-tidy` 诊断。  
- `--completion-style=<value>`  
  代码补全建议的粒度。  
  - `detailed`：为每个语义上不同的补全生成一个补全项，包含完整的类型信息。  
  - `bundled`：将相似的补全项（如函数重载）合并，尽可能显示类型信息。  
- `--fallback-style=<string>`  
  当未找到 `.clang-format` 文件时，默认应用的 `clang-format` 样式。  
- `--function-arg-placeholders`  
  当禁用时，补全仅包含函数调用的括号。当启用时，补全还包含方法参数的占位符。  
- `--header-insertion=<value>`  
  在接受代码补全时添加 `#include` 指令。  
  - `iwyu`：使用“Include What You Use”规则。为顶级符号插入所属头文件，除非该头文件已直接包含或符号已前向声明。  
  - `never`：在代码补全时从不插入 `#include` 指令。  
- `--header-insertion-decorators`  
  在补全标签前添加一个圆形点或空格，表示是否会插入 `#include` 指令。  
- `--import-insertions`  
  如果启用了头文件插入，则在接受代码补全或修复 Objective-C 代码中的包含时添加 `#import` 指令。  
- `--limit-references=<int>`  
  限制 `clangd` 返回的引用数量。`0` 表示无限制（默认值为 `1000`）。  
- `--limit-results=<int>`  
  限制 `clangd` 返回的结果数量。`0` 表示无限制（默认值为 `100`）。  
- `--project-root=<string>`  
  项目根目录的路径。需要设置 `remote-index-address`。  
- `--remote-index-address=<string>`  
  远程索引服务器的地址。  
- `--rename-file-limit=<int>`  
  限制符号重命名时影响的文件数量。`0` 表示无限制（默认值为 `50`）。

---

### **4. 杂项选项 (clangd Miscellaneous Options)**
- `--check[=<string>]`  
  隔离解析一个文件，而不是作为语言服务器运行。用于调查或复现崩溃或配置问题。使用 `--check=<filename>` 尝试解析特定文件。  
- `--enable-config`  
  从 YAML 文件中读取用户和项目配置。  
  - 项目配置来自项目目录中的 `.clangd` 文件。  
  - 用户配置来自以下目录中的 `clangd/config.yaml`：  
    - Windows：`%USERPROFILE%\AppData\Local`  
    - Mac OS：`~/Library/Preferences/`  
    - 其他：`$XDG_CONFIG_HOME`，通常为 `~/.config`。  
    配置文档见：https://clangd.llvm.org/config.html  
- `-j <uint>`  
  `clangd` 使用的异步工作线程数量。后台索引也使用相同数量的线程。  
- `--pch-storage=<value>`  
  将预编译头文件（PCH）存储在内存中会增加内存使用，但可能提高性能。  
  - `disk`：将 PCH 存储在磁盘上。  
  - `memory`：将 PCH 存储在内存中。

---

### **5. 协议和日志选项 (clangd Protocol and Logging Options)**
- `--log=<value>`  
  写入标准错误的日志消息的详细程度。  
  - `error`：仅错误消息。  
  - `info`：高级执行跟踪。  
  - `verbose`：低级详细信息。  
- `--offset-encoding=<value>`  
  强制指定用于字符位置的偏移编码。这会绕过通过客户端能力进行的协商。  
  - `utf-8`：偏移以 UTF-8 字节为单位。  
  - `utf-16`：偏移以 UTF-16 代码单元为单位。  
  - `utf-32`：偏移以 Unicode 码点为单位。  
- `--path-mappings=<string>`  
  在客户端路径（远程编辑器看到的路径）和服务器路径（`clangd` 在磁盘上看到的文件路径）之间进行转换。逗号分隔的 `<client_path>=<server_path>` 对列表，第一个匹配的条目将被使用。例如：`/home/project/incl=/opt/include,/home/project=/workarea/project`。  
- `--pretty`  
  漂亮地打印 JSON 输出。

---

### **总结**
`clangd` 是一个功能强大的语言服务器，提供了丰富的选项来配置其行为，包括：
- 代码补全、符号索引、诊断、头文件插入等功能。
- 后台索引、远程索引、线程优先级等性能优化选项。
- 日志、路径映射、偏移编码等协议和调试选项。

通过这些选项，你可以根据项目需求和工作环境灵活配置 `clangd`，以获得最佳的开发体验。





