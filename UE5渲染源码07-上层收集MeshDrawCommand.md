之前讲了RHI层面和D3D12层面，对渲染命令的封装。今天讲上层逻辑对这块的封装。

把绘制相关的命令，和整个涉及到的相关信息收集起来，形成一个通用的数据结构，然后转换成RHI层面命令的调用。

## FMeshDrawCommand收集场景信息

![image-20240204165154106](Image/UE5渲染源码07-/image-20240204165154106.png)

上层主要就是这个类。

主要收集：

整个渲染管线的所有设置，即pipelinestate，光栅化，stencil，vsps着色器等。

![image-20240204171143854](Image/UE5渲染源码07-/image-20240204171143854.png)

比如这里，着色器相关的shaderbinding，顶点的，索引的。

![image-20240204171249402](Image/UE5渲染源码07-/image-20240204171249402.png)

比如从vsbuffer点进去，都是些简单的数据结构，然后就放的RHI的buffer，这就和RHI对应上了。

![image-20240204172601458](Image/UE5渲染源码07-/image-20240204172601458.png)

然后这就是pso的所有信息。

下面是一些三角形的信息。

等等......

这个数据结构，FMeshDrawCommand就包含了所有的渲染所需的信息，实际上UE的渲染，一个关卡场景，就是把所有的模型、组件的这些信息收集起来。

包括那些静态、动态模型，都会收集起来生成它。

## 收集好后SubmitDraw

![image-20240204172909441](Image/UE5渲染源码07-/image-20240204172909441.png)

收集了场景所有的信息后，就通过这个submit转译到RHI层面的命令调用。

![image-20240204173339332](Image/UE5渲染源码07-/image-20240204173339332.png)

它是个static函数，里面主要就是2步。

begin就是全部设置进去，end就是真正的draw。

![image-20240204173454832](Image/UE5渲染源码07-/image-20240204173454832.png)

## end是真正的draw

先看end，就直接是调RHI的DrawIndex，这就没什么好说的。

然后begin，就是全部的设置和转换，包括pso，stencil，vs输入、ps，uniformbuffer、sampler、srv、纹理图片texture等。

![image-20240204173655011](Image/UE5渲染源码07-/image-20240204173655011.png)

## begin setpso

我们看begin，比如上来就是pso。

![image-20240204173730896](Image/UE5渲染源码07-/image-20240204173730896.png)

![image-20240204173752255](Image/UE5渲染源码07-/image-20240204173752255.png)

![image-20240204173813285](Image/UE5渲染源码07-/image-20240204173813285.png)

![image-20240204173858526](Image/UE5渲染源码07-/image-20240204173858526.png)

pso里面就包含了这些信息，整个渲染管线的所有的相关的数据。

![image-20240204174019985](Image/UE5渲染源码07-/image-20240204174019985.png)

commandlist缓存的信息，填充到pso里面去（？）commandlist当前的一些状态，拿出来。

![image-20240204174243133](Image/UE5渲染源码07-/image-20240204174243133.png)

最后就setgraphpipelinestate。这个是在地图渲染里见过的。

![image-20240204174633148](Image/UE5渲染源码07-/image-20240204174633148.png)

就是调RHI的setpso。

![image-20240204174701772](Image/UE5渲染源码07-/image-20240204174701772.png)

然后就到context的setpso，这就到平台层面的pso了。

## begin set stencil vs等

![image-20240204174812275](Image/UE5渲染源码07-/image-20240204174812275.png)

上面set pso是第一步，然后就是stencil，vs数据等，通过set stream source。

![image-20240204175012175](Image/UE5渲染源码07-/image-20240204175012175.png)

![image-20240204175110758](Image/UE5渲染源码07-/image-20240204175110758.png)

然后最后这个，setoncommandlist，点击去就能看到，和shader相关。

所以一共就是这些，上层把这些收集起来传到RHI，然后RHI层面又有一套传到D3D12，那边又是通过一个上次讲的statecache收集起来，最后一次性执行。

## FMeshDrawCommand怎么来的

![image-20240204175628981](Image/UE5渲染源码07-/image-20240204175628981.png)

有这么个buildmeshdrawcommand函数，根据收集到的FMeshBatch的数据，这个函数讲其转换成meshdrawcommand。

1. 收集meshbatch。
2. 转换成meshdrawcommand。

![image-20240204175859124](Image/UE5渲染源码07-/image-20240204175859124.png)

一个mesh可以有很多个command，它先收集公共的。比如stencil，三角形状态，以及pipelinestate，vertexfactory顶点工厂等，很多类似的公共的。

![image-20240205161230347](Image/UE5渲染源码07-上层收集MeshDrawCommand/image-20240205161230347.png)

真正的创建是在下面的这个for循环。

为什么这是个for循环?

![image-20240205161345264](Image/UE5渲染源码07-上层收集MeshDrawCommand/image-20240205161345264.png)

因为这里一个meshbatch是一组element。虽然多数时候，element只有一个。

这一组element，是共享了vertexfactory，和meterialrenderproxy材质相关的东西。

也就是这一组meshbatch，是装着有同样顶点工厂和材质的element，所以前面会有收集shared的说法。

![image-20240205162404238](Image/UE5渲染源码07-上层收集MeshDrawCommand/image-20240205162404238.png)

对于每一个element，都创建一个FMeshDrawCommand，然后再单独去收集一些信息。vs、ps、gs等。

最后finalize，完成创建。

## FMeshBatch怎么来的

![image-20240205162757939](Image/UE5渲染源码07-上层收集MeshDrawCommand/image-20240205162757939.png)

这里有个比较熟悉的FSceneRenderer类，这个gather函数，就会去收集FMeshBatch。

这个收集，就是真的在一个pass里去收集。

这个FSceneRenderer类，是有很多子类的。比如平时听到的basepass。

看这个函数，我们输入有视口信息，场景信息，全局的buffer，最后就是通过那个collector收集。

![image-20240205163117878](Image/UE5渲染源码07-上层收集MeshDrawCommand/image-20240205163117878.png)

collector里面就有这些东西。

![image-20240205163142744](Image/UE5渲染源码07-上层收集MeshDrawCommand/image-20240205163142744.png)

![image-20240205163153709](Image/UE5渲染源码07-上层收集MeshDrawCommand/image-20240205163153709.png)

我们看下收集过程，先清理，然后做一个view视口剔除，不满足就不收集。

![image-20240205163246158](Image/UE5渲染源码07-上层收集MeshDrawCommand/image-20240205163246158.png)

然后到这，我们去找一个具体的pass看。

![image-20240205163325931](Image/UE5渲染源码07-上层收集MeshDrawCommand/image-20240205163325931.png)

比如这个base pass。

![image-20240205163355994](Image/UE5渲染源码07-上层收集MeshDrawCommand/image-20240205163355994.png)

显示遍历所有的view，一般只有一个view。

![image-20240205163416405](Image/UE5渲染源码07-上层收集MeshDrawCommand/image-20240205163416405.png)

![image-20240205163526587](Image/UE5渲染源码07-上层收集MeshDrawCommand/image-20240205163526587.png)

DrawBatch里，先去分配一个meshbatch，大多数时候一个meshbatch就一个element，前面说过。

这里就是收集起来，add进去到collector收集器。

从这可以大致看到收集的各种信息。

## 总结

1. 了解FMeshDrawCommand，上层的command命令封装，收集后会去调RHI层面的command，然后去调D3D12层面的command。
2. FMeshDrawCommand是通过FMeshBatch收集转换而来。
3. FMeshBatch在FSceneRenderer的gather而来。

