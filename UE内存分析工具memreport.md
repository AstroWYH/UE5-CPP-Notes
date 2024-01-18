UE5 提供了多种Profile内存(Memory)的手段。
memreport -full
在cmd命令行输入 memreport -full
内存报告文件会输出到Saved\Profiling\MemReports\XXX.memreport
用文本文件打开XXX.memreport
## 物理内存和虚拟内存相关信息
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/15b69651-50a7-43ce-b9f6-b595e285464e)
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/e3e71719-8ae3-48f5-bdc1-719ba10f98d2)
## 标记使用内存
UE Allocator对分配的内存打特殊标记, 比如TEXTUREGROUP_Lightmap，TEXTUREGROUP_UI等等。
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/26092b2e-c8c2-4539-b8b7-61e7d98e622d)
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/104450e6-f95f-4183-a927-0e2b06d8c0c6)
## UObject对象数量和内存统计 
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/22b66128-ee99-48a7-a5f9-0b186a1a79a8)
## 最后一行算了Objects的总内存
 ![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/b60f49a0-af2e-45b8-ba41-631a5d4f60dc)
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/3332d83b-af55-4143-a7fd-959a138b2b53)
## RHI内存
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/f77ddb8a-0f94-4631-bbf4-935f45a3176f)
## RHI资源内存(比RHI内存更具体)
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/aa9e3ca9-97c0-43a3-bfbc-e83b30b07b69)
## SkeletalMesh的资源 Object对象数量和内存统计 
 ![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/07d389cd-4682-4a5b-9432-fc178e8e849c)
## StaticMesh的资源 Object对象数量和内存统计
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/c7cdac45-3c31-4c2f-85ff-0e744e4c0057)
## 具体纹理对象内存占用(非VirtualTexture)
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/405a4280-cc39-45f8-ba5f-253354ab018a)
## 粒子系统占用内存 
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/fcf031e9-f2e4-450c-afeb-eee67de8a898)
## .ini配置文件占用内存
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/6b28e4bf-4b1a-4aa5-8278-c506091da30e)
## RT池占用内存 
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/44de899d-7a33-4d6b-8df1-7c32677be68e)
## 具体纹理对象内存占用
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/7958aba9-d09e-4425-bdd2-20924d9577a0)
## StaticMeshCompoent占据内存Object对象统计
 ![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/49bd7ac4-2d30-418c-b22e-6b3c09f1d33d)
