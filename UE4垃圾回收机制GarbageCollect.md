这篇是转载的知乎，后续自己完整梳理一遍。

参考链接：https://zhuanlan.zhihu.com/p/448412111
参考链接：https://www.cnblogs.com/kekec/p/13045042.html

概述：
在C++开发中，其强大且自由的内存处理机制，让我们对内存的管理更为自由，但是同样也带来了烦恼，一旦管理不好自己申请的内存就会造成内存泄漏，且不易察觉。因此有了垃圾回收这种自动内存管理机制，当某个程序占用的内存不会再被该程序访问时，这部分内存就应该被垃圾回收机制回收。

UE4对所有的UObject对象提供了自动垃圾回收机制，当达到GC条件时(内存不足，到达GC时间间隔，切换场景强制GC)，会通过扫描系统中所有的UObject是否存活，来清理那些不需要UObject对象，释放其内存空间。

UE4垃圾回收的核心是使用了标记-清理算法，标记指的是它以所有在ROOT上的UObject列表为根，去递归遍历所有这些根Uobject的引用链，所有能访问到的UObject就标记为可达的。清理阶段会在标记完后，收集所有这些UnReachable Uobjects，对其进行清理回收。标记流程在一帧内完成，而清理阶段可以分为多个帧，时间切片方式完成，所以GC机制的性能点在于标记阶段，如果有太多的UObject对象，那么游戏GC时会有明显的卡顿，所以需要控制内存中UObject数量。
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/db553165-0eb6-49ba-a579-4db629f5b958)


纳入被自动GC机制管理的条件：
UObject对象中的UObject*成员变量必须被UPROPERTY宏定义修饰，自动被加入到引用。
UObject对象实现了派生的AddReferencedObjects接口，手动将UObject*变量添加到引用。
对于Struct结构体中引用的UObject，必须继承FGCObject对象，并实现派生的AddReferencedObjects接口，手动将UObject*变量加入引用。


一个UObject不会被GC的条件：
UObject对象被加入到Root根上(调用AddRoot函数)。
直接或者间接被根节点Root对象引用（UPROPERTY宏修饰的UObject*成员变量 注：UObject*放在UPROPERTY宏修饰的TArray、TMap中也可以）
直接或间接被存活的FGCObject对象引用


手动清理一个UObject：
手动对UObject对象调用MarkPendingKill()，该函数会 将UObject对象对应GUObjectArray中的FUObjectItem的Flags加上EInternalObjectFlags::PendingKill标记，在下一次GC时，会被GC机制在标记阶段，标记为UnReachable对象，进而被回收，同时会自动更新所有引用该对象的UObject*变量为nullptr。



UE4 GC发生的时机：
1.切换场景强制GC:

在加载场景LoadMap时，UE会进行一次完整的GC，清理上个场景无用的对象，释放内存，切场景时进行的是强制GC，强制GC时清理阶段不会分帧进行(全量清理，阻塞主线程)，同步一次性完成清理。

2.定时GC：

UE会定时清理无用的UObject，默认是60s调用一次GC，该数值可以在游戏设置中修改
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/43d83025-363e-4529-a5eb-562d44be955b)


如果在游戏中设置了低内存阈值，控制变量gc.LowMemory.MemoryThresholdMB，当游戏可用内存低于这个阈值时，定时GC的频率会加快，默认每30s进行一次GC。定时GC的清理阶段会分帧进行(增量清理)，尽量降低对游戏性能的损耗，标记阶段依旧会在同一帧进行。

3.手动进行GC:

在我们想要UE立刻进行垃圾回收时，我们可以通过调用GEngine->ForceGarbageCollection(bool bForcePurge)接口，来让UE进行一次完整的垃圾回收，当bForcePurge为true时，会更改强制GC标识，进行的是强制GC，为false时会更新时间戳，触发定时GC。它们的区别在于强制GC进行全量清理，而定时GC进行增量清理。

UE4 GC流程:
GC入口
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/d11ac609-c0ed-4b0f-b09c-2c94cf7146a4)

GC开始时需要获取GC锁，在GC期间禁止其它线程对UObject进行操作，在GC结束后释放锁，CollectGarbageInternal是进行GC操作的真正入口。

GC操作主要分为两个流程，标记阶段与清理阶段。



标记阶段
入口代码：
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/fca16bd4-c79d-4b96-84d0-cbb1ed1b66ea)


主要流程代码分析：
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/b8c09271-e3fa-4ba7-82a6-8a8283065fe4)


首先要确保FGCObject::GGCObjectReferencer这个UObject对象加入到ObjectsToSerialize列表中，所有不需要被GC的UObject都需要加入到这个ObjectsToSerialize列表中，以这个列表为根，来遍历引用链，进行标记。对于Struct结构体引用的Uobejct指针变量，如果想加入到GC机制必须继承FGCObject类并实现派生接口AddReferencedObjects，其原理就是在这里将它的一个static UObject* GGCObjectReferencer加入到了引用遍历列表，在后面标记时会遍历调用我们自己实现的AddReferencedObjects接口，将引用的UObject的对象加入到ObjectsToSerialize列表中递归标记。



标记阶段主要分为两个部分

![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/44baa9d8-03a5-499c-8e00-5a9c49201d4f)
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/6cdab503-2ff8-4fc2-b95f-dc879ec9206a)


标记不可达，将所有不确定是否可达的对象都标记为不可达。
可达性分析，将所有确定可达的对象都清除掉不可达标记。
一．标记不可达流程主要代码分析：
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/858d6569-f127-411d-b115-618e2313350d)


标记不可达操作是多线程操作，所以先确定有多少个允许被GC的UObject数量，有多少个WorkerThreads，然后确定每个线程分配多少个UObject进行标记操作。然后分配NumThreads个ObjectsToSerializeArrays用来存储临时标记对象的队列，最终汇总到一个ObjectsToSerialize中。


多线程遍历GUObjectArray，这段代码主要做了两个事情：
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/81fb6be6-e430-4dc3-8a50-07d95efc446f)

将所有Root上的UObject加入到ObjectsToSerialize中，后面需要根据这个列表来递归遍历进行可达性分析。
如果开启了集群特性bWithClusters，并且这个在Root的UObject也是集群中的一员，那么把该UObjectItem加入到KeepClusterRefsList这个列表中，表示该UObject所在集群被Root引用，表示可达，后面会继续处理这个列表。
注：集群，Cluster，可以把它当作一个有着共同生命周期的UObjects集合，把它当作一个整体，该整体的可达性由这个集群的Cluster Root是否可达来决定，如果Cluster Root不可达，那么表示该Cluster中所有的UObject都是不可达的，将会被垃圾回收机制全部回收(也有例外，如果被标记了ReachableInCluster会被略过)。该特性可以加快垃圾回收标记流程，在集群中的UObject不会递归遍历引用链，只有Cluster Root是关键。

Cluster在生成时，就是以它的Cluster Root的UObject为根，递归遍历该对象的引用链(TokenStream)，将所有引用的UObject都加入到了Cluster中。

如图：
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/b792e517-2d2a-4e6e-90db-109c63b0009d)


上图就是执行完后标记的样子，继续看代码：

![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/6bf02249-4f25-4878-a090-cb7fba4deb43)
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/77744182-ce19-48ea-bfb7-fe6637609ed4)


处理了没有在Root上情况，主要处理不在Cluster中的普通UObject或者 在Cluster中的Cluster Root UObject，分3情形：

如果没有被标记为PendingKill并且包含了传入的KeepFlag标识，那么该UObject不可被GC，认为是可达的。如果该UObject在Cluster中，表达集群可达，那么也需要加入到KeepClusterRefsList中。
如果该UObject是Cluster的Root UObject且被标记为PendingKill，那么需要将该UObject加入到ClustersToDissolveList，后面会将该ClustersToDissolveList列表中的Cluters及其ReferencedByClusters递归解散，并标记成不可达，其目的是为了实现自动更新引用，后面会详解。
除以上这些情况，其它所有的UObject都会标记为不可达，因为不确定是否可达，所以都标记为不可达，后面会做可达性分析去清理掉可达对象的不可达标记。
注意：这里并没有处理在Cluster中非Cluster Root的UObject，因为对于在Cluster中的非Cluster Root成员，它们的可达性并不关键，最后收集的时候只会根据Cluster Root的可达性来判断是否可达，所以没做处理，加快流程，但是默认是Reachable的！

标记流程如图：

![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/7197c0a7-e978-4dd4-aeb3-284265b71437)

继续看代码：

![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/fe9e7a1a-c663-4a49-a2bd-b13d75dea093)


多线程标记完后将多个线程收集的ObjectsToSerialize达列表汇总到一个ObjectsToSerialize中。
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/99f86e53-4a26-4f03-9478-49cc3978234f)

处理所有在集群中有处于PendingKill状态UObject成员的Cluster，它会递归的解散所有引用该Cluster的Clusters，并将所有Cluster中的Uobjects标记为不可达。解散的目的是为了实现自动引用更新，Cluster中有UObject被标记PendingKill，引用这个Cluster的Cluster中必然有对象指向了PendingKill的对象。因为标记PendingKill的对象，所有引用它的指针变量需要自动重置为NULL。自动更新引用需要后面在可达性分析流程时，递归遍历引用链来实现，但是Cluster中成员是不会递归遍历引用链的，所以只能解散Cluster并标记不可达，按照普通UObject对象来进行垃圾回收。

注意：Cluster虽然可以引用Cluster，但是它不会像UObject那样，引用链很长，一个Cluster的引用链不超过2，啥意思呢，举例，ClusterA引用了ClusterB，ClusterB引用了ClusterC，我们正常理解ClusterA->ClusterB->ClusterC，但是实际在创建ClusterA时，因为它引用了ClusterB，那么UE会把ClsuterB的所有引用加入到ClusterA的引用中。所以正确的图示应该是：

![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/c8b2f787-b390-4e7e-a87c-3f3e4640054d)

![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/4b119752-1dee-46c7-a546-d9b925a3a488)



接下来就会遍历处理上面提到的KeepClusterRefsList列表，该列表中的Cluster中都存在UObject成员在Root上或者标记了KeepFlags标识，主要分为两种情况：

如果非Cluster Root成员在Root上或标记了KeepFlags，那么将该Cluster普通成员标记ReachableInCluster，并清理掉该Cluster的Cluster Root成员的Unreachable标记，即可达，然后将该Cluster直接引用的所有Clusters的Cluster Root，以及引用的MutableObjects标记为可达。如果引用的Cluster或MutableObjects存在PendingKill，会解散当前的Cluster(为了自动更新引用)，并将所有的成员标记可达，加入到ObjectsToSerialize当作普通对象处理。
如果是Cluster Root成员，那么直接将该Cluster直接引用的所有Clusters的Cluster Root，以及引用的MutableObjects标记为可达。如果存在PendingKill，处理同上。
注意：Cluster也是可以引用其它的Cluster的，它也可以引用其它的UObject，正常一个Cluster引用的所有UObject都应该加入到这个Cluster中，但是有的UObject不可以成为Cluster成员，所以就放在MutableObjects存储。

总结：这一步的目的就是将所有的Cluster，只要Cluster中成员有在Root上或有KeepFlags标识，那么这个Cluster是可达的，它所引用的Clusters与UObjects也是可达的。

如图：

![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/054a277d-282a-4aee-bc76-2e6efc83eef9)



二．可达性分析：

代码入口：
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/7982f0cb-3522-434b-bde5-0c33538d4313)
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/52ab04b7-e477-4180-b2c3-1a691d325ca2)



可达性分析主要做的事情是将上面收集的ObjectsToSerialize列表，递归遍历其UObject引用链，所有引用到的均标记为可达。可以多线程进行遍历ObjectsToSerialize，加快标记速度，无论是否多线程操作，最终都会转发调用到ProcessObjectArray。

ProcessObjectArray主要代码分析：
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/9d8ffd44-7584-496d-8349-72abda166c93)


ProcessObjectArray主要遍历了ObjectsToSerialize，然后获取了每个UObject的ReferenceTokenStream，这是每个Uojbect为了加快GC，根据自身UClass反射机制，遍历所有的FProperty，将需要纳入GC管理机制的Property，提前按照自定义的格式生成一个Token流，这个Token流实际就是UObject对象的引用链。这也是为什么需要纳入自动GC的UObject需要带有UPROERTY标识，只有这样才能被反射机制找到加入到Token Stream中。

Token Stream简单图解：


![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/34bc8847-f046-45c5-8a41-ca333247093a)

每个Token Stream都是一个FGCReferenceInfo队列，在FGCRerenceInfo中，存放了每个属性的类型与偏移地址，可以据此获取到属性变量。

Token Stream生成接口：

![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/e9a4ae79-ded0-4afd-b127-96f83cf42818)


该接口主要遍历UObject的反射属性，为每个属性生成FGCRenerenceInfo，并加入到Token Stream中，同时会递归生成父类对象的Token Stream，最后将父类的Token Stream加入到当前UObject对象的Token Stream前面。

处理FGCReferenceInfo，进行可达性分析：
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/a81d4689-ce1f-4ea8-8582-0c22be71b09b)


FGCReferenceInfo对象类型有很多，这里只列出两种情况的进行可达性分析，除此之外还有很多情况，如引用的Tarray，Tset，Tmap等如果包含了UObject类型，也会进行可达性分析，其中原理大同小异，可自行展开，接下来看这两种情况：

一． FGCReferenceInfo是Object类型，即引用了一个UObject*变量：

根据FGCReferenceInfo的类型与偏移地址，获取到引用UObject*变量的引用，对该UObject进行可达性进行分析：

函数入口：
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/a022f624-0f8d-4bfc-a7ef-3d913a501779)


主要流程代码分析，主要分3中情况：

1.引用UObject处于PendingKill
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/ca61c744-f410-441f-98a0-2abbf50e7276)


引用的Uojbect已经处于PendingKill状态，要自动将引用该UObject的变量重置为NULL，这就是自动引用更新。

2.引用UObject不可达

![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/5124773b-494a-4afa-bc29-27589e404d1e)

当引用的Object不可达，那么就需要将其状态重置为可达的，如果是普通UObject那么会将它加入到ObjectsToSerialize列表中，后面会递归它的引用链进行可达性分析。如果该Object还是一个Cluster的Cluster Root，那么应该把该Cluster引用的所有Cluster Root与MutableObjects都置为可达。

注意：这里没有处理如果引用的这个UObject是Cluster中的成员怎么办，那是因为一个Cluster 中的UObject默认是Reachable的，它永远不会进这里，所以这里我们可以知道，一个Cluster中的所有对象，都是不会进行对其引用链进行可达性分析的，只关注Cluster Root的可达性，这样就加快了GC标记速度。

3.引用UObject可达

![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/91ebbf69-d7f2-4445-bfe8-33e5925415e7)

如果一个UObject是可达的，如果是普通对象或者Cluster Root那么说明标记过了，不会进这里，这里只会处理引用的这个UObject是Cluster普通成员(因为普通成员默认Reachable)，如果Cluster普通成员被引用了，说明该Cluster应该保留，所以会将Cluster Root置为可达，并将Cluster引用的Clusters的Cluster Root与MutableObject都置为可达。

至此，就完成了一个引用UObject*变量的可达性分析，可达性分析前后图解：


![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/56fddcd6-0e75-4612-b317-f15caffd34e9)


二． FGCReferenceInfo是AddReferencedObjects类型，即引用了一个UObject*变量：

这里就是实现当一个普通的结构体继承了FGCObject就可以hook住UObject不被垃圾回收，是因为FGCObject中的UGCObjectReferencer对象，它派生实现了AddReferencedObjects，并在生成Token Stream时将该函数指针存进去。UGCObjectReferencer在最初手动的加入到了ObjectsToSerialize列表中，这里进行调用它的AddReferencedObjects，最终会转发调用到我们自己实现的FGCObject对象的AddReferencedObjects，会把我们所有想要不被GC的UObject都加入到ObjectsToSerialize列表中，后面继续递归遍历引用链进行可达性标记。



至此，标记阶段的两个注意步骤，标记不可达，与可达性分析就完成了，接下来继续看清理阶段。



清理阶段
主要分为3个步骤，收集，清理，析构

1.收集：
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/84493cca-74d6-4ab8-afc4-e58821a2c7ba)

在标记阶段完成后，UE会把所有的不可达的Uobejct收集到一个GunreachableObjects列表里，为了方便后面清理，收集代码入口：


主要流程代码分析：
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/fb714417-443b-49c4-9ab3-c1672c690a50)


多线程收集，遍历GUObjectArray所有的UObject，如果该Uobejct不可达就加入到GunreachableObjects(这里不会将Cluster的普通对象加入，普通对象默认可达)，如果该Uobject是一个Cluster Root，那么将该Uobject加入到ClusterItemsToDestroy队列中，后续会继续收集不可达的Cluster。
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/58825b61-6595-434a-b98a-7751a4d18589)


遍历ClusterItemsToDestroy，将这些不可达的Cluster中的所有成员标记不可达，并加入到GunreachableObjects列表中，这里可以看出，一个Cluster中的成员是否被回收，它们自身是否可达并不重要，完全取决于Cluster Root是否可达。

收集图解：
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/c52780e6-7b28-4ee4-8005-c3813eb9f3c6)


2.清理
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/335be454-cc4a-4f70-b70c-982da20c2714)

清理和析构在同一个接口中实现，接口：


该接口实际就是增量清理，如果设置了增量清理bUseTimeLimit为True，那么每一次的清理操作时间超过TimeLimit时会返回，等待下一次的Tick帧到来时继续清理，增量清理默认是每次清理不超过2ms，如果bUseTimeLimit为false，那么进行全量清理，会一帧内清理直到所有的不可达对象全部销毁。

清理主要流程分析：
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/1f47c2fb-4a3b-47b9-979d-50161de6d2d9)


UnhashUnreachableObjects接口会遍历GunreachableObjects列表，对所有的UObject调用ConditionalBeginDestroy。
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/9f900668-fb99-469c-b8a2-bd895fca7f0f)

IncrementalDestroyGarbage中会遍历GunreachableObjects列表，对所有的UObject调用ConditionalFinishDestroy。

3.析构：

对所有的不可达的UObject分别调用了ConditionalBeginDestroy和ConditionalFinishDestroy后，开始对所有的不可达对象进行析构，销毁对象。

代码入口：
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/1e35a5c3-9dad-4958-9aa5-9da08edd60bf)


在TickDestroyObjects中，会遍历GunreachableObjects，对Uobejct对象析构销毁，并释放内存，如果增量回收，达到时间后会返回，下一次tick帧继续销毁。



至此，整个垃圾回收机制的流程就完成了，主要对标记与清理流程进行了分析，很多细节没有详细展开，包括多线程回收，可自行展开。



GarbageClollect优化：
UE为GC机制提供了优化设置
![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/a18d4f53-9358-44c1-8955-08b1d04c0944)


Allow Parallel GC：开启多线程GC，默认开启，开启后，GC流程的标记阶段会多线程进行标记，可以加快GC。

Incremental BeginDestroy Enabled：开启增量BeginDestroy，默认开启，开启后，清理阶段对UObject进行BeginDestroy可以时间切片，增量进行，降低GC对性能影响。

Mutithreaded Destruction Enabled：开启多线程析构，默认开启，开启后，清理阶段对UObject析构销毁时，会进行多线程处理，只有对支持多线程析构的对象会在多线程处理，如Upackage。不支持的对象依旧在主线程销毁。可以加快GC

Create Garbage Collector UObject Clusters：开启Cluster，默认开启，开启后会将适合的对象加入到Cluster，Cluster中的对象在标记阶段不会递归遍历引用链进行标记，加快标记流程，加快GC。

Asset Clustering Enabled：默认开启，开启后，加载的资源将包含的actors创建Cluster，加快GC。

Actor Clustering Enable：默认不开启，开启后，加载的关卡资源将包含的actors创建Cluster，加快GC。

Blueprint Clustering Enabled：默认不开启，开启后，加载的蓝图类，将引用创建Cluster，加快GC。

Use DisregardForGC On Dedicated Servers：默认不开启，在DS上禁用DisregardForGC

Minimum GC Cluster size：Cluster最小成员数量，低于该数量不创建Cluster

Maximum Object Count Not Considerd By GC：不被GC的UObject数量，默认值是1，对于引擎启动初始化时创建的UObject，以及创建的Native对象的UClass，这些对象都是需要常驻内存，不需要被GC掉的，设置这个值后，这些UObject将不会被GC流程遍历标记清理，加快GC。且设置后引擎初始化时会为GUObjectArray初始分配该数量的空间，避免频繁申请内存，加快启动。该数值可以在启动日志中获取：

![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/54f9b651-dd73-402c-8eab-eaba1bc7fc96)

将该值写回编辑器配置中。

Size Of Permanent Object pool：不被GC的UObject总内存占用，引擎初始化时会为GUObjectAllocator提前申请该数值的内存空间，所有初始化过程中创建的UObject直接在该内存空间存储，避免频繁申请内存，加快启动。

![image](https://github.com/AstroWYH/UE5-CPP-Notes/assets/94472801/4c45a784-2462-4469-9925-507304eba561)

Maximum number of Uobjects that can exist in cooked game：独立游戏模式下UObject存在的最大数量，超过该值时，非shipping会触发check并提示。

Maximum number of Uobjects that can exist in the editor：编辑器模式下UObject存在的最大数量，超过该值时，非shipping会触发check并提示。

参考链接：https://zhuanlan.zhihu.com/p/448412111
参考链接：https://www.cnblogs.com/kekec/p/13045042.html
