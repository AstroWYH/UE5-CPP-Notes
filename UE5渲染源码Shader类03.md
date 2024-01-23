## RHI层面的shader类

![image-20240123172456202](Images/UE5渲染源码Shader类03/image-20240123172456202.png)

FRHIShader，继承自FRHIResource。思路也跟之前一样，这个类内容简单。其子类D3D12和OGL会有具体复杂的实现。

![image-20240123172733002](Images/UE5渲染源码Shader类03/image-20240123172733002.png)

![image-20240123172742681](Images/UE5渲染源码Shader类03/image-20240123172742681.png)

这个Frequency枚举，用于定义该Shader是哪个类型。

![image-20240123172837522](Images/UE5渲染源码Shader类03/image-20240123172837522.png)

其子类，可以看到常见的顶点、像素着色器，但在这个层面的类定义依然非常简单。就是一些初始化操作。

![image-20240123173019946](Images/UE5渲染源码Shader类03/image-20240123173019946.png)

以顶点着色气为例，进入到它D3D12的实现，可以看到依然很简单。

其实它的主要内容，是放到多继承的那个FD3D12ShaderData里面的。

![image-20240123173044388](Images/UE5渲染源码Shader类03/image-20240123173044388.png)

shader最重要的就是它的代码Code

![image-20240123173245559](Images/UE5渲染源码Shader类03/image-20240123173245559.png)

内部将其转换成D3D12的bytecode。

![image-20240123173414291](Images/UE5渲染源码Shader类03/image-20240123173414291.png)

怎么创建一个shader呢，就是上面这个函数。

![image-20240123173452915](Images/UE5渲染源码Shader类03/image-20240123173452915.png)

进去后就会发现，也是定义在DynamicRHI里面的，之前说过dynamic里有一大半的渲染基本操作。

实现，会在平台真正的子类去实现。

![image-20240123173623995](Images/UE5渲染源码Shader类03/image-20240123173623995.png)

比如OGL。

## OGL创建shader

![image-20240123173716426](Images/UE5渲染源码Shader类03/image-20240123173716426.png)

进入到OGL的创建shader，先会解析一下Code，因为UE这里的code不是纯代码，还有会带有参数。所以会先用一个Reader解析一下。

![image-20240123174013241](Images/UE5渲染源码Shader类03/image-20240123174013241.png)

![image-20240123174049719](Images/UE5渲染源码Shader类03/image-20240123174049719.png)

这个解析的内部，可以看到Code前半段是代码，后半段是Optional的数据。代码长度=总长-OptionalData长度。

![image-20240123174201152](Images/UE5渲染源码Shader类03/image-20240123174201152.png)

在后面的OptionalData数据段里，提供了根据key去找对应的data。

![image-20240123174356946](Images/UE5渲染源码Shader类03/image-20240123174356946.png)

然后继续刚才的创建shader流程，这里会根据不同平台，对glsl进行转换。

![image-20240123174437395](Images/UE5渲染源码Shader类03/image-20240123174437395.png)

![image-20240123174501897](Images/UE5渲染源码Shader类03/image-20240123174501897.png)

![image-20240123174516767](Images/UE5渲染源码Shader类03/image-20240123174516767.png)

![image-20240123174536104](Images/UE5渲染源码Shader类03/image-20240123174536104.png)

最终层层深入，找到OGL的编译shader的代码。

![image-20240123174637746](Images/UE5渲染源码Shader类03/image-20240123174637746.png)

## D3D12创建Shader

然后看D3D12的shader创建过程，这个就简单很多，因为不涉及到其他平台的转换等操作，比较直接。

![image-20240123174802954](Images/UE5渲染源码Shader类03/image-20240123174802954.png)

![image-20240123174913113](Images/UE5渲染源码Shader类03/image-20240123174913113.png)

当然也有Reader解析代码和额外参数。然后进入initshadercommon，获取到代码Code。这里就直接结束了。为什么看起来没有编译环节？

![image-20240123175022931](Images/UE5渲染源码Shader类03/image-20240123175022931.png)

因为前面提到过，D3D12的shader代码其实主要在FD3D12ShaderData的Code里面，刚才的Init获取了代码，那么在要用的时候直接GetShaderBytecode转换用就行了。

12:06