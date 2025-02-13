对于C#程序

从C:\Users\<Username>\AppData\Local\CrashDumps获取Core文件

通过VS2022打开

打开后，找到右侧选项，打开设置PDB和EXE等符号库的位置

注意：PDB和EXE必须和出core的完全对应，即使代码一样重新编译的都不行，否则看不到符号

必须要之前编译的那一版，符号才能看到正确的

![image-20250102111322563](Images/VS2022CoreDump调试/image-20250102111322563.png)
