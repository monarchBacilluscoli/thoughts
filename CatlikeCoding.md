# CatlikeCoding问题和想法手记

- [CatlikeCoding问题和想法手记](#catlikecoding问题和想法手记)
  - [一些信息](#一些信息)
- [Chapter1](#chapter1)
  - [Game Objects and Scripts](#game-objects-and-scripts)
  - [Building a Graph](#building-a-graph)
  - [Switching Between Functions](#switching-between-functions)
- [Chapter2](#chapter2)
  - [第二课讲课内容](#第二课讲课内容)
  - [Measuring-Performance](#measuring-performance)
  - [Compute Shader](#compute-shader)
  - [Animating a Fractal](#animating-a-fractal)
  - [Organic Variety](#organic-variety)
  - [第二次课程笔记](#第二次课程笔记)
- [Unity Shader入门精要](#unity-shader入门精要)
  - [渲染流水线](#渲染流水线)
  - [Unity Shader基础](#unity-shader基础)
- [Chapter3](#chapter3)
  - [Sliding a Sphere](#sliding-a-sphere)
  - [Physics](#physics)
  - [Surface Contact](#surface-contact)
  - [Orbit Camera](#orbit-camera)
  - [Custom Gravity](#custom-gravity)
  - [Complex-Gravity](#complex-gravity)
  - [第三次课程](#第三次课程)
    - [问题](#问题)
    - [笔记](#笔记)
- [Chapter 4](#chapter-4)
  - [Moving the Ground](#moving-the-ground)
  - [Climbing](#climbing)
  - [Swimming](#swimming)
  - [Reactive Environment](#reactive-environment)
  - [Rolling](#rolling)
  - [课程笔记](#课程笔记)
- [Charpter 5](#charpter-5)
  - [Persisting-Object](#persisting-object)
  - [Object-Variety](#object-variety)
    - [Reusing-Objects](#reusing-objects)
    - [Spawn Zone](#spawn-zone)
    - [More Game State](#more-game-state)
  - [课程笔记](#课程笔记-1)
  - [课程问题](#课程问题)

-------------------------------------------------

## 一些信息
1. 客户端作业交给孙艺/段瑞
2. 常量需要首字母大写
3. 工程和代码注意：
   1. 去掉不必要的using
   2. 命名
   3. 注释
   4. addictive add scenes

# Chapter1

## Game Objects and Scripts
1. Multi-Scene editing的意义是什么？多场景协作？
    除了多场景协作，文档中还说了“allows you to create large streaming worlds”，这个在应用Multi-Scene时是怎么样的实践呢？
2. “Mesh Filter”指的是什么？Filter的意义是什么？
3. 注意有的时候Position偏向可能是挂在别的物体底下了。
4. 使用Parent Empty Object来进行定位的时候就如同处理了坐标系
5. `class`的modifier是`public`和`external`。
6. Microsoft.Unity.Analyzers 网页的旁边有download package
7. 新的警报已经变成了is missing the class attribute 'extensionofnativeclass'，所以这意味着什么？
    > Unity can only use subtypes of MonoBehaviour to create components.

    1.所以`Monobehaviour`到底做了什么？我们不继承它的时候意味着什么？——有这个问题的原因是我看到了一个知乎上的问题，它的意思似乎是能够不用mb就不用它（为了解耦引擎/提升效率什么的），那这样编程的逻辑是什么呢？如果进行Update等等操作呢？通过哪里调用它呢？可以实现吗？难道是一个Game Manager在游戏开始去调用一个Manager的脚本，然后这个脚本完全自己实现了Timer和Update以及控制所有对象的逻辑？
       1. MonoBehaviour is the base class from which every Unity script derives. It offers some life cycle functions that are easier for you to develop your app and game.[https://stackoverflow.com/questions/41630986/what-is-monobehaviour-in-unity-3d](来源)
        就是我把本来引擎做的事情拿来自己实现了？似乎也可能哦。
    2. 那这里体现了什么思想呢？似乎有个说法叫**基于组件开发**，这指的是什么呢？
        1. 它和OOP不是一个层级的内容，相对更高一点，不是OO那种时刻都要使用的东西。
8. Gimbal Lock是什么？一直都没弄懂
   1. 似乎指的是以陀螺仪万向节作为欧拉旋转基础设施的情况下出现的在最后一次旋转的时候无法向某些分量旋转的情况
   2. 其中的一些关键在于：
      1. 万向节中的某个轴必须相对world或者父物体静止，如果它也跟着物体动了那么这个物体和世界就没什么联系了
      2. 万向节具有嵌套关系
      3. 万向节的问题并非在于“接下来不能旋转”，而在于表示方式非线性、万向锁的存在要求麻烦的旋转顺序处理，以及不同旋转顺序造成的结果不同等等..
9.  Single Precision的telepotaion trick是指的什么？
10. 为什么我的GameObeject没有材质，我明明拖了材质到上面啊。为什么我需要从Mesh Renderer上拿到材质？
    1.  这里可能有图形学的基础理解。
11. `const`和`readonly`差异是什么？

## Building a Graph
1. 所以Prefab能够统一改什么？不能够统一改另一些什么？如果在scene中修改了呢？
   1. 直接回答第三个问题——如果scene中修改的东西，修改Prefab就不管用了
2. 给了一个`Transform`到`Instantiate`，创造了一个`GameObject`？仍然返回一个`Transform`？
3. `while`内部定义的变量可以被重复使用？
4. 值类型也必须显式初始化才能使用？
5. 值类型的字段到底是不可被修改还是装箱的值类型字段不可被修改？
6. 什么是shader？
   1. "The GPU runs shader program to render 3D objects"
7. value type直接用field传回和用property传回有差别？为什么？
8. Time.time的使用，Mathf的使用
9. Shader的语法和逻辑是怎样的？

## Switching Between Functions
1. 修改code似乎可以让别的也跟着修改
2. 学习到的内容：
   1. `delegate[]`，使用index
   2. `enum`在editor indicator的显示
   3. `enum`和`delegate`联动
3. 双层`for`和一维数组结合使用的时候i可以在内层++，就完全避免了计算i的开销
4. 这些分型的东西实在是太让人头秃了。
   1. 我知道了，这些分形让人比较头大的原因可能是因为我没有图形学**uv坐标**方面的知识

# Chapter2

## 第二课讲课内容
1. Code Review希望review的是真正的逻辑修改而非编辑器的修改。

## Measuring-Performance
1. 为什么Measuring Performence中提到"In my case it suggests that the entire frame took **51.4ms** to render, but the statistics panel reported **36FPS**, matching the render thread time. The FPS indicator seems to takes the worst of both and assumes that matches the frame rate...**only takes the CPU side into account**"
   1. emmm也是，两条threads应该都是CPU threads，然后两条threads应该是并行的。所以使用了最差的那个作为简单的估计。
2. Our graph contains 10.000 points, so it appears that each point got rendered three times. That's once for *a depth pass*, once for *shadow casters*—listed separately as well—and once to render the final cube, per point. The other three batches are for additional work like the sky box and shadow processing that is independent of our graph. There were also six set-pass calls, which can be though of as the GPU getting reconfigured to render in a different way, like with a di!erent material.
   提到了给到GPU的三次东西：
   1. **depth pass**
   2. **shadow caster**
   3. final cube
   这三次东西到底是什么呢？
   1. pass好像是指“一次顶点和像素shader，且有一些设置伴随”。
   2. 具体depth/shadow可能要看看图形学了
3. **set-pass**又是什么呢？
   1. 简单点讲是“切换材质球”[来源：知乎](https://www.zhihu.com/question/299878086/answer/520612546)
4. **DRP（默认渲染管线）、URP（通用渲染管线）、SRP（可编程渲染管线）**都是什么呢？
   1. 什么是渲染管线？
      1. 基本上就是几何-光栅化-像素这个流程[来源：细说细说图形学渲染管线](https://positiveczp.github.io/%E7%BB%86%E8%AF%B4%E5%9B%BE%E5%BD%A2%E5%AD%A6%E6%B8%B2%E6%9F%93%E7%AE%A1%E7%BA%BF.html)
   2. 这些差别是什么呢？
   3. 那么这些名词是以什么样的形式存在的实体？哪些仅仅是概念，而哪些是实际的包？哪些仅在Unity中使用？
5. URP启动后图形材质变洋红色，需要升级材质 [CSDN](https://blog.csdn.net/zhenghongzhi6/article/details/99437048)。
   1. Unity会提示不兼容
6. URP dynamic batching完全不起效，连数值都没差
7. 在DRP中启动dynamic batching数值上起效，但是帧数不体现
8. URP - SRP batching + GPU instancing +10fps
9.  URP - SRP batching +za2FPS
10. 什么是**延迟渲染**？
    1.  出发点似乎是为了应对**前向渲染**的多光源重复渲染的问题
11. URP和DRP的渲染效果似乎不同。
12. Profiler的每帧用时似乎和Statistics的对应不上啊？Profiler是40多，statistics说是十几
    1.  嗯，在Build之后的数据中对上了，没有Editor Overhead之后就没毛病了
13. UI中的stretch和non-stretch的设置选项是不一致的
    1.  stretch是Left Right Top Bottom
    2.  non-stretch是PosX posY Width Height
    3.  说白了就是一个是绝对位置和大小，一个是相对位置和大小
14. 为什么TMPUGUI会需要一个[`SerializeField`](https://docs.unity3d.com/ScriptReference/SerializeField.html)？
    1.  什么是`SerializeField`？这个Serializable Atrribute还不太一样
        1.  "When Unity serializes your scripts, **it only serializes public fields**. If you also want Unity to serialize your private fields you can add the SerializeField attribute to those fields."
        2.  这个使用的是Unity的序列化系统，并非.net的，并且不能序列化任何静态field和属性
        3.  关于热重载：更改并保存脚本时，Unity 会热重载所有当前加载的脚本数据。它*首先将所有可序列化变量存储在所有加载的脚本中，并在加载脚本后恢复它们*。热重载后，*所有不可序列化的数据都将丢失*。——[脚本序列化](https://docs.unity3d.com/cn/2018.4/Manual/script-Serialization.html)
        4.  所以发生了什么？**存储在所有加载的脚本**中指的是？具体是存储到哪里去了？为什么是Serialize而不是Keep或者Remain、Store之类的attribute
        5.  然后它为什么又会在Indicator中显示？当然这个好像解决了，因为使用它的原因就是为了修改、热重载、执行
15. `Time.deltaTime`和`Time.unscaledDeltaTime`的区别？
    1.  很简单，前者是可以被`Time.timeScale`给影响的，而后者是真实时间。
16. 所以`float`、`int`等我是需要对其初始化的嘛？不需要，默认为0，但是不初始化会报错。
17. `enum`定义到`class`内外有什么差别呢？
18. It is important to be aware of memory allocation for temporary objects and eliminate recurring ones as much as possible. Fortunately SetText and Unity's UI update only perform these memory allocations in the editor, for various reasons, like updating the text input field. If we profile a build then we won't find these allocations. So it's essential to profile builds. Profiling editor play mode is only good for a first impression. 这句话是什么意思？
19. 并且我在Profiler中没有看到这些内存分配啊
20. 为什么要有那么多`private`的非要设置成`SerializeField`才行，不能直接public嘛？
    1.  注意前面关于热重载的介绍

尚未解决：4.2（URP、DRP），**14（SerializeField，热重载）**

## Compute Shader
1. 什么是Single matrix？为什么是它？
2. 定义kernel function的时候指定的numthread和Dispatch定义了什么？参数的意义是什么？为什么是三维向量形式？和GPU的架构有什么关系嘛？Dispatch的group又是什么？
   1. group的定义是分辨率/8向上取整...也对，我们画100分辨率的东西需要计算10000个方块，然后每个计算group已经是`8*8*1`了，所以就直接`/8`就很好算
3. "hot reload"是什么？
   1. 看前面*热重载*
4. HLSL语法是什么？
5. SerializedField跟在editor上显示有啥关系？
   1. 看前面*热重载*
6. `someField = default`指的是什么？
7. Compute Shader和c#代码是如何协同的？举例：DrawMeshInstancedProcedural()是如何与shader协同的？
8. shader的manu label是什么？target 4.5是什么？
   1. 就是最上面的`Shader "Custom/FractalSurfaceGPU2"`Custom是在shader选择列表中的一级列表
9.  PBR graph似乎只能和URP compitable
10. ComputeBuffer是由C#脚本来控制申请大小的而shader实际上并不能做这些事情对吗？ComputeBuffer的构建在读入-处理-输出中的位置是什么？它存在何处？为什么不是shader去申请然后c#去get呢？
    1.  所以shader是GPU的逻辑代码咯？GPU实际没有办法去主动和CPU通信？
11. 为什么要用`UNITY_PROCEDURAL_INSTANCING_ENABLED`
12. Transforming Matrix - unity_ObjToWorld是一个全局变量吗？为什么这里需要定义这个？。
    1.  是
13. Unbelieveable！真的可怕！帧数直线上升
    1.  Graph2.Update要22ms，但GPUGraph.Update只需要0.02-0.03ms了！天哪
    2.  为什么不管是Render还是...哦哦哦那个是CPU的render thread
    3.  理解为什么会产生这样的差异->需要理解GPU的特点->可以帮助理解compute shader的代码逻辑（比如numthread）+帮助理解图形学的内容——需要Unity Shader入门精要
14. 图形方向好像有问题，并且也很闪烁 
15. Asynchronous shader compilation为啥和procedural drawing不匹配？
16. ProceduralDrawing到底是什么？
    1.  似乎是读abtray data from buffer，而并非vertex和面——[Graphics.DrawProcedural](https://docs.unity3d.com/ScriptReference/Graphics.DrawProcedural.html)
17. 要想让画面动起来，记得在editor中正确配置空对象对代码和shader的引用
18. 所以`SV_DispatchThreadID`是从Dispatch->Group做二级换算而来的具体的Thread的ID([MicrosoftDoc](https://docs.microsoft.com/en-us/windows/win32/direct3dhlsl/sv-dispatchthreadid))
19. PointSurfaceGPU适用于DRP，PointURPGPU适用于URP
20. 后来它怎么改的就可以动态改变绘制数量并且不会让部分方块留存在屏幕？
    1.  原来是申请**初始分辨率**平方的buffer，绘制的时候**初始分辨率**平方的buffer
    2.  上面两者造成了一个问题就是所有buffer都会被绘制，即使其已经不用了；并且没办法构建更多的buffer。
    3.  后面就改成申请**最大分辨率**，并且只绘制**当前分辨率**的内容。
    4.  注意Step的更新会导致虽然fixed掉的point位置不变，但是大小仍然会随着绘制逻辑的更新而变大
21. 我去，宏的问题真的好神奇。折行宏后面不能加注释。另外出错之后太几把难调试了。
    1.  比如定义的“WaveKernal”编译器死活就是找不到，后来发现是Kernel写成Kernal了！
22. 为什么kernel要和buffer绑定？——We also have to set the positions buffer, which doesn't copy any data but links the buffer to the kernel...Its first argument is the index of the kernel function, because a compute shader can contain multiple kernels and buffers can be linked to specific ones.
    1.  SetBuffer函数中提到"Buffers and textures are *set per-kernel*. Use FindKernel to find kernel index by function name".——[ComputeShader.SetBuffer](https://docs.unity3d.com/ScriptReference/ComputeShader.SetBuffer.html)
    2.  后面又说In this case we don't have to provide a kernel index for the bu!er.

未解决问题：**2(numthread, dispatch)**，10，**22(为什么buffer要link到kernel)**

## Animating a Fractal
1. 为什么
   ```csharp
    child.transform.localPosition = 0.75f*Vector3.right;
    child.transform.localScale = 0.5*vector3.one;
   ```
   这两句即可保证所有的sphere变成1/2大小并且相互touching？
   1. `localScale`在父子之间会传递
   2. `localPosition`的比例会受ancestor的`localScale`的影响
2. 为什么Clone `Component`的时候连`GameObject`也会被Clone。并且All child objects也会被Clone
3. 左右手坐标系及每个轴的旋转正方向的判定：
   1. 食指y拇指x，其余三指和这个平面90°就是z
   2. 左手伸出就是左手，否则是右手
   3. x手坐标系就伸x手握住轴的正方向，四指指向就是旋转正方向
   from [判断三维坐标系旋转正方向的简单方法](http://wonderffee.github.io/blog/2013/10/17/a-simple-method-to-determine-positive-rotation-in-in-three-dimensional-space/)
4. 如何理解“The recursive hierarchy of our fractal with all of its independently-moving parts is something that Unity struggles with. It has to update the parts in isolation, calculate their object-to-world conversion matrices, then cull them, and finally render them either with GPU instancing or with the SRP batcher”
   1. 喔，一个一个对象分别独立更新Update实际上是一个比较慢的事，如果可以由Manager控制来进行flaten的更新，那将非常好。
5. 如何理解default？
6. 如果constructor的参数是空的，那么我们在Initializer的时候可以省略掉括号。
7. 为什么在设定移动距离的时候需要localScale？parent已经设定了并且scale也已经设定了啊（它应该是复杂化了，但可能为了后面做准备）
8. parent和child的旋转如果要stack需要设定顺序注意。搞定四元数乘法
9. 对一个物体进行旋转就是在物体的旋转上*一个四元数
10. Procedural Drawing的意思应该就是前面说的“读取任意数据并绘制”，而非CPU给它顶点
11. 三维空间的变换矩阵（各种）[三维变换中的矩阵](https://zhuanlan.zhihu.com/p/147282442)
12. `Matrix4x4.TRS()`意为Transalation、Rotation和Scale
13. 四元数相乘的连续微小误差意为什么？
14. 为什么使用ComputeBuffer通常要用`OnEnabled()`？意味着ComputeBuffer在hot reload的时候会发生什么？
15. `OnValidate`在inspector被改变的时候被调用，或者在undo、redo的时候被调用。
    1.  文档中并没有介绍两者(`OnValidate`/`OnEnable`)的相对order
    2.  但是运行表明`OnValidate`比`OnEnable`要早一些
16. shader graph中为什么要有那个custom function去添加hlsl之中的函数呢？
    1.  原文中有一句“We'll use a Custom Function node to include the HLSL file in our shader graph. The idea is that the node invokes a function from the file. *Although we don't need this functionality, the code won't be included unless we connect it to our graph*.So we'll add a properly-formatted *dummy function* to PointGPU that simply passes through a float3 value without changing it.”
    2.  似乎意思是就想加入这个文件而已
17. 最后`MaterialPropertyBlock`的作用是什么啊？
18. 非仿射变换是啥玩意？
19. 使用shader graph的时候记得save assets
20. SIMD了解一下
21. ReadOnly特性：
    1.  The ReadOnly attribute lets you mark a member of *a struct* used in a job as read-only.
    2.  表明这个字段是可以被安全地parallel访问的
22. Job要和Burst Compilation合并使用
    1.  Burst is specifically optimized to work with *Unity's Mathematics library*, which is designed with vectorization in mind.
23. using static指的是什么？
24. 我笑了，我的URP和DRP的显示效果不一样hhh
    1.  URP的问题是在于把localPosition写错成worldPosition了

未解决问题：15,17,

## Organic Variety
1. shader graph和代码中的In Out是怎么联系起来的
2. 只有4-component vectors才能被送到GPU
3. 为什么找一个旋转过的东西的up方向是`mul(mul(parent.worldRotation, part.rotation), up())`？
   1. 其实很合理就是旋转之后你是知道一个旋转的状态(角度)，但是方向，你还是需要叠加一个vector，就是这样。
4. 为什么从world up到local up的叉积是旋转轴？
   1. 就是这样。并且这个旋转如果是正数那么方向就应该是从前者到后者。

未解决问题：

## 第二次课程笔记
1. window - framedebugger 可以打开图形的debugger，enable之后可以打开GPU所有的命令
2. Compute shader是CPU是上一帧装给buffer，这一帧计算，计算是在所有的绘制步骤之前的
   1. 你在shader graph中的所有步骤都在这个debugger里面都可以有显示，并且顺序是按照依赖关系进行的
3. 同步是两点：一点是CPU给了GPU，然后就是GPU内部流水线就是直接处理依赖关系的。如果GetBuffer在CPU之中是会block掉CPU的

# Unity Shader入门精要
## 渲染流水线

## Unity Shader基础
1. 所以generated code是为表面着色器生成的顶点/片元着色器
2. 结构就是三个部分：
   1. Property
      1. 就是一些可以显示在面板里面的变量
   2. SubShader
      1. 最少一个
      2. 
   3. Fallback

# Chapter3 
## Sliding a Sphere
1. TrailRenderer真是个有意思的东西
2. `SerializeField`做了什么？
   1. 告诉editor save 并 expose 这个变量，从而它可以在editor中被修改
   2. 并且在Hot Reload的时候可以被存储并被重新应用
3. 这个直接控制velocity和acceleration的混合模式是这样一个思路：
   1. 分每个轴进行
      1. 我可以直接控制velocity所以我有一个最终速度
      2. 我同时还需要一个加速度的逻辑，加速度在这里不用input处理了，input处理的是速度，这里直接用最大加速度*deltaTime
      3. 所以就让没有达到最终速度的时候采用加速，达到的时候就不进行加速了
   2. 恒定的加速度和最大速度下，直接控制速度，但通过手动调节固定加速度大小来调节手感。还是相当于一个速度控制器，但是有了固定加速度了，就有了惯性的感觉。

## Physics
1. 在`Edit-Project Setting-Time-Fixed TimeStep`中可以修改`FixedUpdate()`的更新速率。
2. 为什么物理模拟最好放到`FixedUpdate()`里面
   1. 这个是最能服人的说法——[Unity为什么推荐在FixedUpdate处理物理模拟？](https://zhuanlan.zhihu.com/p/137395596)
3. `|=`和`GetButtonDown()`的协同作用
   1. 如果对这个按键的行为还没有resolve那么bool值将保持true直到被resolved，而不是在下一帧直接被false
4. 调用顺序 `FixedUpdate()`->`OnCollisionXXX()`->`Update()`
5. `Collision.GetContact()`可以找到所有的碰撞体
6. 为什么`EvaluateCollision()`中是`|=`
7. 为什么快速按下跳跃可以跳的更高？
   1. 画个图就知道，因为这里是直接给的速度
8. 为什么要用一个字段去记录m_velocity而不是直接用m_body.velocity?
   1. 早忘了
9.  在Unity中创建的对象必须要变成prefab才能当作asset导出？
10. In play mode the *sphere instances aren't kept synchronized* with the prefab. You'd have to select the spheres and change their max ground angle directly.
11. 注意这里有一个非常经典的范式就是“存储当前状态以便在帧与帧之间进行增量更新”。
    1.  还需要处理好函数调用顺序间的问题
12. 计算速度的时候比较挠头，问题还是在：
    1.  对向量点乘掌握不清晰
    2.  思路被情绪引导
    它的核心就是，把沿水平方向的投影到平面上

## Surface Contact
1. The physics step during which the sphere gets launched still has a collision. We act on that data during the next step, so we think that we're still grounded while we no longer are. It's the step after that when we no longer get collision data. So we're always going to be a bit too late, but this isn't a problem as long as we're aware of it.
   1. 这一段什么意思？上一帧的物理检测不是在这一帧会被刷新吗？并且Update在物理检测之后啊
2. 使用法线的单位向量的分量来判断各种角度
3. 有一个现象，就是最大爬升角度在prefab这种修改之后不会被应用
   1. 发现问题出在定义了角度，然后在OnValidate之中做了dot的计算但是判定的时候没有应用这个dot
   2. 并且最关键的是，判定的时候使用的是角度值，没有*转换成弧度*
4. 这里有一个编程的范式，就是`UpdateState()`，应该还有别的思路在里面但是还没有全部figure out
5. 注意跳跃的第一帧仍然会判断OnGrounded的现象，应该是按下时+1，但是此时没有应用跳跃速度没有起飞
   1. 上一帧Update之中按下跳跃并desire
   2. 下一帧fixedUpdate更新速度（不知道应用了没，也不知道是不是真正离地了）
   3. 再下一帧有了离地速度并且又过了一帧原则上应该离地
6. 鸡贼，为了处理台阶的不平滑移动，把台阶的碰撞体给拉直了
7. Stair的grounded角度又不灵了
8. 同一个物体，两种碰撞模式：
   1. 对Agent而言台阶需要平滑移动
   2. 对台阶上的小物体而言，台阶要像是台阶。
   3. *为什么设定了两层layer就会搞定这个问题？*
9. 注意将ask的使用方式输入空间可以由一个Transform来设定，这样就可以绑定到任意本地坐标如何讲本地坐标转换为世界坐标？
   `mask&(1<<layer)`这样就能得知是否mask中包含layer
10. 为什么要检测墙呢？
    1.  因为这里有一个不合理的情况，就是我明明是在一个夹角里，但是我却因为不是踩的Ground而不能跳，只能air jump，但是air jump又用完次数了，于是乎就get stuck
11. 在墙跳之后要revisit之前的air jumping。也就是行为之间的相互影响注意
12. more than one step after a jump，非常关键，因为jump（实际上是fixedUpdate之中的速度改变）是下一帧进行的

## Orbit Camera
1. camera在LateUpdate中进行，避免在视野中的对象偏移
2. 一些手段：
   1. target到focus的反向插值
   2. (1-c)^t的平滑拉动
      1. 以及避免平滑拉动的无限插值的阈值设定
3. 旋转，需要加入初始方向，才能得到目标方向
4. 注意`Transform`的`forward`和`Vector3`的`forward`应该不一样
   1. trans: Returns a normalized vector representing the *blue axis of the transform* in world space.
   2. vec: Shorthand for writing Vector3(0, 0, 1).
5. 别忘了总是要乘speed以及deltaTime
6. 输入空间可以由一个Transform来设定，这样就可以绑定到任意本地坐标
   1. 如何将本地坐标转换为世界坐标？
7. well. Most importantly, we should ignore the sphere itself. When casting from inside the sphere it will always be ignored, but a less responsive camera can end up casting from outside the sphere. If it then hits the sphere the camera would jump to the opposite side of the sphere.

## Custom Gravity
1. 自定义重力实际上就是+一个重力方向的`Vector3`然后在用的时候带入计算
2. 至于控制，则加入控制轴的根据重力的变换即可
3. 静态类是如何得知当前场景的重力信息的
   1. 当前是“场景”还是“appdomain”
4. 不同的ForceMode有什么差别呢？
5. 注意闲置的时候关闭重力计算
   1. When a Rigidbody is *moving slower than a defined minimum linear or rotational speed*, the physics engine assumes it has come to a halt. When this happens, the GameObject *does not move again until it receives a collision or force,* and so it is set to “sleeping” mode.[`isSleeping()`](https://docs.unity3d.com/Manual/RigidbodiesOverview.html)

## Complex-Gravity
1. `List`的`Remove()`是如何实现的？
   1. “Removes the *first occurrence* of a specific object from the List<T>.”
2. `Debug.Assert`用法
   1. If the first argument is false, it logs an assertion error, using the second argument message if provided. *The third argument is what gets highlighted in the editor if the message is selected in the console.*
   2. 只会在Developement build之中有效
3. unity中的向量应该是列向量，因为Quaternion在前。


## 第三次课程
### 问题
1. 为什么物理更新需要放到`FixedUpdate()`之中？
   1. 虽然说这个函数会按照固定时长更新，但是说实话我没法相信这个hhh11
   2. 我何时应该用固定步长，何时应该用变长步长

### 笔记
1. 一般都是移动
   1. 运动移动
   2. 寻路移动
      1. staring path ? 应该是点集连接的实现

# Chapter 4
## Moving the Ground
1. Rigidbody.isKinematic(运动学？)Controls whether physics affects the rigidbody.*If isKinematic is enabled, Forces, collisions or joints will not affect the rigidbody anymore*. The rigidbody will be under full control of animation or script control by changing transform.position.
2. Animation可以和物理同步
   1. `Animator.UpdateMode = Animate Physics`
3. 真的，我的天，rotation对于一个对象来说不视作transform的。你如果检测那个transform，他在rotate，你不会get到一点点transfoProject Setting里的Physics Layer Collision Matrix很有用rm
4. 把Animation拉成线性：
   1. Curve显示，`ctrl+a`，右键选择key frame，双侧linear就行了
5. 所以PhysX和`FixedUpdate()`是谁先调用的？
   1. 在 `FixedUpdate()` 之后将立即进行所有物理计算和更新。[Unity Doc](https://docs.unity3d.com/cn/2018.4/Manual/ExecutionOrder.html)
   2. 在 `FixedUpdate()` 内应用运动计算时，无需将值乘以 `Time.deltaTime`。这是因为 FixedUpdate 的调用基于可靠的计时器（独立于帧率）。
6. 不要让任何有`Rigidbody`的个体成为任何`Rigidbody`的child

## Climbing
1. 注意stair，这个layer设置错了sphere就可以直接爬墙了
   1. 把墙当作stair
   2. Project Setting - Physics Layer - Collision Matrix很有用
2. 为啥`CheckClimbing()`时把contact、normal等都设置和ground相同？

## Swimming
1. 所以Trigger不被检测的原因是什么？
   1. 一般是怎样的物体才会作为Trigger？
   2. 所有的水都是Trigger
2. 别的Mask都是1，为什么水的Mask是0？
3. `Collision`和`Collider`的区别
   1. `Collision`包含`Collider`，嗯
      1. 初次之外Collision还包含一些`impact`
4. Unity的layer的内部实现，为什么不能和layerMask直接&。
5. OnXXXEnter都是在Physics阶段，但是在FixedUpdate调用之后。
   1. Trigger会早于Collision
6. 我觉得水阻力这部分有点问题
   1. 似乎它在假设一个可行范围，但是很奇怪，10就ok而多一点都不行，公式如下：
   ```csharp
   if (InWater) {
      velocity *= 1f - waterDrag * Time.deltaTime;
   }
   ```
7. 查一下C#的四则运算的类型问题
8. 妈的这些逻辑是不是能用状态机去写啊
9. 对于速度的影响有几种：
   1.  做加减法（acceleration）
   2.  根据比例缩放（drag）
   3.  直接消除
10. 为什么AddForceAtPositionf到非全局位置会导致旋转？
11. 将物体在水中稳定住：
    1.  设定浮力作用点和重力受力点的偏移
    2.  如果是大的物体记得设置多于一个的平衡受力点

## Reactive Environment
1.  为什么把`JumpPhase`设为-1就可以解决`SnapToGround`的问题？
    1.  因为这样就欺骗`SnapToGround`让他以为刚刚起跳，这个时候肯定不能用Snap把？
2.  加速的典型用法是：
    1. `Mathf.MoveTowards(velocity.y, m_speed, m_acceleration * Time.deltaTime)`
3.  C# Event的修饰符是`public`，但是在Unity中的`UnityEvent`则不用
4.  `OnTriggerExit`没法在物体deactiveted的时候被调用
5.  检测碰撞一节使用了`collider.gameObject.activeInHierarchy`，这是否意味着此处存在“子物体active但父物体unactive”的情况？
    1.  对，并且物体被disable和collider被disable是两个东西
    2.  但是这里`!collider`的检查似乎不可用，应该是`!collider.enabled`
6.  To avoid needlessly invoking FixedUpdate continuously we can disable the component when it awakens and after the last collider exits. Then we only enable it after something enters. This works because the trigger methods always get invoked, regardless whether a behavior is enabled.
    1.  很有趣，这里有一个现象就是当物体处于disabled的时候OnTriggerXXX仍然能工作？
    2.  对，估计是因为OnTriggerEnter是注册给Collider组件的，所以这个函数的从属组件是否失效并不影响，影响的只是FixedUpdate这些
7. 检查是否为hot reload的方法：如果处于editor并且所有层级物体都enable则是
8. 他这里说了一个使用Animation是OK的，但是有的时候简单的两点移动用interpolation就行了
9. Unity无法序列化泛型事件类型，所以需要将该类型变为具体类型，即从
   ```csharp
   [SerializeField]
   UnityEvent<GameObject> onValueChanged = default;
   ```
   派生为
   ```csharp
   [System.Serializable]
   public class OnvalueChangedEvent:UnityEvent<GameObject>>{}

   [SerializeField]
   OnValueChangedEvent onValueChanged = default;
   ```
   1. 估计这里的原因是泛型类型序列化是C#不支持
10. `MovePosition()`和直接给`Position`赋值有何差异？
11. 这个Interpolator和Slider分开并用事件触发的机制可以思考一下
    1. “*How that value gets changed* is a separate issue from the interpolation itself. Keeping the slider separate also makes it possible to use it for *multiple interpolations* .”
12. 脚本名中间不能有空格，否则提示找不到
13. 是否smooth可以用一个lambda和trigger来设定
14. 如果两个将要碰撞的碰撞体，比如Silder和Ground之间有其他物体，那些物体可能爆开可能穿模，解决之道：
    1.  碰撞面有角度，其他物体可以被稳定挤出
    2.  碰撞面不会完全碰到一起

## Rolling
1. 加速度在两个轴分别生效但是速度变化率一样，这样会导致如果方向不沿着某个坐标轴，加速度使之停止的时候最后速度就会平行于其中一个轴，因为另一个轴的速度很快减到零了
2.  Quaternion.Euler(rotationAxis * angle) * m_ball.localRotation;
   1. 这是一种什么操作？
      1. 哦，Quaternion.Euler的本意就是返回一个四元数，表示在各轴上旋转对应轴上的读数
3. 有意思，做球的旋转只需要
   1. 接触面法线cross移动方向就得到了旋转轴
   2. 距离对半径进行计算就得到了旋转角度
   3. 结了
4. 旋转矫正的这一句是什么意思？
   Quaternion newAlignment = Quaternion.FromToRotation(ballAxis, rotationAxis) * rotation;

## 课程笔记
1. activated和enable是两个东西

# Charpter 5
## Persisting-Object
1. GetKeyDown只是第一帧返回true
   1. GetKey可以保持
2. 原来`System.IO`有自己的`persistentDataPath()`
   1. 以及可以自动处理正反斜杠
3. BinaryWrite和Reader用起来居然这么简单
4. 模式：操，原来`Save` and `Load`自己的功能可以直接写在自己身上
5. 模式：不同的对象的重载可以完全是不同的逻辑，比如同一个类型，Game存整个游戏，PO只存自己
   1. 很奇怪，这里Storage的Save&Load接收PersisitableObject然后调用它自己的Save&Load给他writer&reader...很搅合
      1. 这里的逻辑就是存有PS的PO（Game）调用了PS.Save(this)去存储另一些PO（Prefab）
      2. 上面的PS.Save(this)在内部调用的是Game对PO的重载，这个重载不像那些PO一样只存储自己，而是存储整个游戏。
      3. 太迷了，读取任何游戏对象的时候要构建那个SaveReader并传入，然后SaveReader构建的时候就需要传入文档之中读取的版本号
         1. 于是只能某一类有这个SaveReader的内部BReader，在构造器中先对这个存档进行读取然后传回当前对象...嗯
         2. 至于调用这个构造器的，那应该是一个管理类
            1. Storage和Game的关系就是：
               1. G调用S，传入G
               2. S调用G，传入S.writer
6. Instantiate可以接受任意自定义类型，然后只要Prefab上有这个组件，就能赋值给这个组件类型的变量并由Instantiate初始化这个prefab
7. 
## Object-Variety
1. script可以是组件，也可以是一种Asset！
   1. 类型继承ScriptableObject——这就是一种asset了
   2. 类型用[CreateAssetMenu]——这就可以在Unty的menu中显示了！
2. [Statistics](https://docs.unity3d.com/cn/2018.4/Manual/RenderingStatistics.html)窗口信息
   * Batches：批处理的批次
   * Saved by batching：合并的批次数量
   * Tris & Verts：三角形和顶点数目
   * SetPass：渲染pass的数量，每个需要一个新的着色器，可能会带来CPU开销
   * Visible Skinned Meshes：渲染的蒙皮网格数量
3. 在Instantiate一个对象的时候，如果对象有一个只需单次设定的id之类，可否使用`readonly`和ctor结合实现这一目标？
   1. 不可，因为在`Instantiate()`对象的时候无法调用ctor
4. `Debug.LogError()`和`Debug.Log()`有何不同
5. 用存档版本号来区分不同存档
6. **`PropertyBlock`到底是啥**
7. `Destroy()`可以用于组件、GameObject或者asset

### Reusing-Objects
1. 奇怪，一个float的property在事件下拉里会有一个dynamic和一个static
   1. That happens when you picked CreationSpeed from the Static Parameters list. As its name implies, that allows you to configure a fixed value to use as the argument, instead of the dynamic slider value. You have to use the dynamic option instead
2. 模式：计时overshoot的时候减去预计值得到overshoot，视作下一次的开始
   1. 然后overshoot可能会超过2，于是乎写一个while循环，如果>=1则一直进行上例
3. 模式：recycling：将某个物体deactivated并将之放到回收池中
   1. 创建的时候再从回收池移除并Activate之
4. 模式：array中删除一个对象——将最末尾对象拷贝到当前对象的位置并删除末尾对象
5. `ScriptableObject`到底有什么屌的使其可以变成asset
6. 命名：shapeToRecycled
7. 把pools序列化好像不是很好，为什么？
   1. Can't we just mark pools as Serializable? That will make Unity save the pools as part of the asset, persisting it between editor play sessions and including it in builds. That is not what we want.
8. 模式：有些物体不必要copy到所有出现的scene中，可以只放在自己的scene并且结合
   1. 灯光是activated的那个scene的灯光
9. `SetActiveScene`和`GetActiveScene`都不能在scene创建的当前帧get到
10. 协程：StartCoroutine
    1.  yield
    2.  IEnumerator
11. 烘焙场景的灯光数据代表了什么？
<!-- ? 了解一下协程 -->
12. Awake gets invoked immediately when the scene is loaded, but doesn't count as loaded yet. Start and later Update invocation happens afterwards, when the scene is offcially loaded.
13. Scene会面临几个问题：
    1.  Load和Get/Set没法在同一帧
    2.  `Awake()`没法get到完全loaded的scene
    3.  load scene之后记得`SetActive()`
14. 有病啊，为什么我要在for循环里进行input检测？
   ```csharp
   void Update () {
		// …
		else if (Input.GetKeyDown(loadKey)) {
			BeginNewGame();
			storage.Load(this);
		}
		else {
			for (int i = 1; i <= levelCount; i++) {
				if (Input.GetKeyDown(KeyCode.Alpha0 + i)) {
					StartCoroutine(LoadLevel(i));
					return;
				}
			}
		}
		// …
	}
   ```
15. 注意这俩的区别
    1.  GetSceneAt
    2.  GetSceneByBuildIndex
        1.  build编号一定要在build中真正有那个scene时才会生成

### Spawn Zone
1. 如果对象的引用跨了scene，Unity将阻止该引用，两个解决方案：
   1. 使用存储引用的对象去遍历引用所在的scene的层次结构
   2. 让引用所在的scene去设定这个property
2. 静态变量在compilation之间不会保存，为了恢复，只好在OnEnable()中重设他们
   1. 过程：
      1. 所有组件disabled
      2. 游戏状态被save
      3. 重新编译
      4. 重置游戏状态
      5. 所有组件enabled
3. 将field从public变为private [serializeField]不会导致挂载的对象失效

### More Game State
1. `Random`是一个静态对象
2. 模式：将当前Level的信息放到静态类全局可见直接调用。
   1. 注意它是怎么就解耦了的
3. **如何调试在Unity中最后才报出来错的内容？**毕竟Unity整个程序在这边断点的时候会挂起，如何让这个时候就在编辑器这边弹出出错信息？
4. `using()`包含的语句中如果使用了协程，并且协程中使用了`using()`中取得的系统资源，那么会导致问题因为：
   1. using中的资源（是文件而非BinaryReader）在括号结束时就会被释放，这样在异时/异处使用它就会导致诸如“Cannot access a closed file”的错误
   2. 解决之道：
      1. 完全不用`using`，而是手动释放。在所有位置都进行释放比较考验耐心和技术
      2. 实践：这个情况是由于读取存档的中间需要加载场景才能继续后面的读档和应用存档数据（这些数据是基于场景的），场景加载比较慢并且需要一帧之后才会真正加载完毕，所以才使用协程。特定于于这个案例，我们可以先将所有数据一次全部读取出来再慢慢处理。
      3. **疑问**：
         1. 这里使用的是先读入内存，然后就完全不用担心。
            1. 那么读入内存指的是？我是否可以一直开启该文件而不释放？如果不能，那影响是什么？文件读取器的Dispatch到底做了什么事情？
            2. 所以唯一的差异就是BinaryWriter的构造器中接受了内存流。那么这怎么就可以了？内存流就可以安全被释放？
5. 模式：可以在某个需要保存的对象的类中设置list，存储所有待存储对象，遍历之并调用其Save方法
   1. UI也要记得更新


## 课程笔记
1. 如果英语词汇有问题GRE、如果长难句有问题GRE长难句阅读解析
2. Unity管理内存的工作变成了管理生命周期

## 课程问题
1. 游戏中启动了多个场景，如果有些物体需要在当这些场景卸载的时候不被析构，应该怎么办？
   1. 现成的方法：
      1. 使用`DontDestroyOnLoad()`，内部机制是将该物体至于额外的*从不卸载的场景*中，并和当前场景是additive的关系。但是需要处理再行addtive添加scene时的多余物体。
   2. 存档到内存（静态化）：
      1. 使用静态字段存储状态
      2. 静态类，将当前场景的所有物体的数据存储到静态类中，然后在新场景开始时重新创建
   3. 存档到文件：
      1. 将相关数据序列化到存档文件并在新场景中重新加载