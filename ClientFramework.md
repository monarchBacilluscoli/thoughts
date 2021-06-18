# BJ客戶端框架

## 問題
1. 我如何才能正確打開這個框架
   1. 給到的trunk到底是游戲工程還是什麽？如何正確打開並使VS正確管理Project從而可以使用自動補全、符號查找之類
2. 导出框架有问题
   1. 因为这已经是ok的项目了

## 配置介紹
1. List的設定爲什麽會有不同：
   1. 複雜的那種難道是就地定義了一個類型？
      1. 对
   2. 簡單的那個就不是就地定義的而是用的別人的？
      1. 对，可以这么理解
2. Name本身是標記的基礎文字所以會被標記為null不需要（12:00）
3. 這些表之間的關係需要瞭解一下 
4. ST(string table)表是干嘛的，多语言翻译是怎么实现的
   1. 首先多语言那一行如果需要翻译，会在该类型中定义一个新的字段，这个字段会被其他的表中的数据给填充
   2. 这个其他的表，就是ST_（语言代号）.xlsl表，每个需要翻译的列后面都会跟一列标注该字段在ST表中的ID的列。根据ID直接上对应语言的ST表查就可以查得到那个翻译
   
### 记忆
#### 枚举表
1. 枚举还是数据看单元格`A2`
2. Data表的第11行是默认值
   1. 第7-11行是list才用
   2. LIST的那种复杂定义形式实际上是就地定义了一个class
3. NONE导出类型是供参考的（比如说明）
4. 数据准备过程
   1. 配置文件分析工具将配置文件生成为两个部分：Code和Data
      1. Code是类型定义orEnum
      2. Data是包含实际数据的bin文件
   2. Unity中BlackJack——Framework——ConfigData——BuildAll会把bin文件拷贝到Assets/Gameproject/RuntimeAssets/ConfigData
      1. 并且Unity会将bin转换成.asset（就是SO），其中保存了bin文件的数据。
         1. 应该是包了一层，用于使Unity可以以二进制数据的方式加载他  
   3. 然后二进制数据加载到内存中，才会有Protobuff反序列化生成真实数据
   4. **然后这些数据可以通过ID来获取（啊？啥的ID，啊，不是那个ST的ID虽然可能那个也可），也就是一次获取数据的一行？那这个ID从哪获取？**
      1. 哦，其实文件里也是分了类的，比如GetConfigDataCharChipSlotInfo()实际上只是从一个单独的存储了CharChipSlot信息的表里查找某个ID的数据而已 

## 资源管理
1. 什么是`WeakReference`——并不增加引用计数，仍然可以被GC `Reclaim()`
2. Bundle和SingleBundle的关系
3. 自动绑定是什么，怎么实现的
   1. 就是创建的时候自动挂载节点

### 记忆
1. 一般实机上使用的是AssetBundle的加载方式
2. LoadAsset()传入的资源路径是自Assets起
   1. ResourceManager中有Cache（<string, AssetCacheItem>），cache中引用资源的是弱引用
   2. 
3. LoadAssetsCorutine()就是加载一组资源，串行的方式
4. 从Bundle中Load就是先查path在哪个bundle，没有就把它下下来先，然后通过加载现场去加载所有的依赖Bundle
5. 整个流程就是缓存直接-缓存Bundle-Bundle下载这个样子。
6. BundleData里面的version从哪拿到的？玩家怎么知道version变化的？
   1. **他似乎说了一个三打不溜什么剋使的地址指定新的版本号**
#### 自动绑定
1. 实际上我自己就在做这个事，只不过我的绑定对象的路径是直接写在逻辑里了的，而这里是通过给一个变量一个带路径的attribute，将这个路径的物件直接绑给这个变量（这个可以实现一波）
2. prefab Controller creator就是将某个Controller动态创建至某个Prefab的机制（又加了一步）
   1. 使用方式是（一个小细节）：使用Creator的静态函数把那个对象传进去然后这个静态函数会解析上面的Creator配置并绑定对应的Controller
      1. 所以这其实是个工具函数，工具函数就静态就好了
3. PrefabResouceContainer用于实现Prefab复用
   1. 哦，通过只是脚本引用而没有直接挂进去的prefab来在创建时自动加载从而实现子prefab更新-挂载他的prefab也更新的效果

## 场景管理
1. `SceneManager`负责管理Layer  
   1. 创建Layer会首先创建一个Layer，此时在hierarchy创建一个Dummy节点`m_uiLayerRootPrefab`
   2. 然后根据传入的资源路径加载资源
   3. 资源加载完毕后会Instantiate并创建之
2. 听说一个显示对象就是一个Layer
   1. 那么真的有那么多摄像机？
      1. 可能那些都是一个摄像机，毕竟还有scene的layer和3D的layer
      2. **摄像机之间都是啥关系**
3. 为什么是UI+3D+UI的结构，也就是为啥是UI在3D之下？
   1. 是这样的，就是3D+UI+3D中间的UI就是指“在3D之下”的那一层
      1. 看来3D的Layer会灵活些？为啥 no，不是，3D应该就只有一层
   2. 现在只支持两层UI中间夹3D，听起来实际完整的结构是（从下到上）：3D+UI+3D+UI(是不是还能+一层3D)
   3. Group2是下层的layer，但是层级位置比较新
4. Push（显示）和Pop（取消显示）层是从unusedRoot取出或加入
5. 叠套的绘制顺序由什么控制？
   1. 首先是`StayOnTop`
   2. 然后是push入栈的顺序
   3. 最最后，非常特殊的情况才是layer的`Priority`
6. 叠套的绘制顺序由什么实现
   1. 物品摄像机的depth值
### 问题相关
UIController、ListUIController和UITask的作用
### 涉及脚本
* `SceneManager.cs`：管理Layer：增删改查适配特效
* `SceneLayerBase.cs`：Layer的名称、描述、状态
* `UISceneLayer.cs`：节点，相机，画布之类property
* `ThreeDSceneLayer.cs`：只有LayerCamera
* `UnitySceneLayer.cs`：继承3DSceneLayer，只是根节点列表

## 音频和PlayerCtx
### 音频
1. 音频管理器如何管理挂载在prefab上的音频？还是通过场景管理器类似的去拿到GO再去管理
   1. 那统一管理呢？
### PlayerContext
1. PlayerCtx是用于在客户端缓存玩家*逻辑数据*的地方
   1. 此外还是封装了和服务器网络通信的地方
2. 前一个说法并不甚确切，PlayerCtx本身是处理协议收发，而其中包含的LogicBlock才是用于存储处理的部分
3. 几种协议类型：
   1. Req-Ack：客户端和服务器一问一答
   2. Ntf：服务器直接发起
4. 所有的On事件回调都在OnServerMessage()里面被resolve
#### LogicBlock
1. 下层有两个东西：
   1. DataContainer
      1. 实际存储数据的集合，跟逻辑无关，只是一个Container
   2. DataSection
      1. DataContainer可能包含过多对象，数据块可能非常大，这个用来将Container拆分，分块更新

## UITask和UIManager   
1. 什么是Task？
   1. 客户端完成功能的集合
2. UITask受UIManager控制
   1. 其中最关键的两个函数
      1. `StartUITask`
      2. `StartUITaskWithPrepare`
         1. 比如显示仓库之前需要从服务器拉取数据，然后才能正常显示
      
3. **Task的Indent是什么**，难道是他的handle？但这里还有Name啊
   1. 感觉是参数信息，所有的参数就往里扔就对了
      1. 基类就俩string，一个taskName一个Mode，然而注意`UIIntentCustom`就对了，这里有一个`Dictionary<string, object>`
         1. 至于UIIntentReturnable就是有一个在Task1启动Task2的时候把他的UIIntent给Task2即可，但注意待返回的那个intent  一定要在stack里面
   2. 其中的string targetMode是啥玩意？
      1. 字符串反正灵活使用吧
4. UI的Register主要的作 用是分组、冲突
5. prepare、redirect什么的都是用来处理接续或者同步的Task逻辑才产生的

### 涉及脚本
1. UIManager：创建、开始、返回、停止、暂停Task
2. Task
3. UITaskBase
4. NetWorkTransactionTask

## UITask更新管线
1. UITask所进行的几个步骤：逻辑数据获取，动态/静态资源的管理，界面刷新逻辑的控制；当然还有一些点击事件等等。
2. 界面刷新过程/`UITask`的一次管线刷新过程：显示出来到动画停止
3. 核心函数：
   1. 所有的`Task`启动、恢复、return等除了数据准备，都会调用同一个`UITaskBase.StartUpdatePipeLine()`函数
      1. 更新数据 ``
      2. 准备资源 ``
      3. 界面刷新 `StartUpdateView()`
         1. 界面刷新甚至从整个数据开始拉取的时候一般会禁止UI交互
         2. 如果是第一次打开，那么进行初始化工作
         3. 然后进行UI的auto bind
   2. `UITaskBase.StartUpdatePipeLine()`会调用`UpdateDataCache()`，会有疑问就是为啥这里也要处理data。
      1. 注意这里是DataCache，就是从PlayerCtx中拿到的数据可能并不符合显示需求，比如你可能需要对装备栏里的东西进行一个过滤，显示的信息是过滤后的信息
      2. 啊？艹之后就是静态资源和动态资源加载...哦资源不是数据
4. `UITask`管理若干个`UILayer`以及所属的`UIController`，负责创建并显示`Layer`，同时通过`UIController`控制`Layer`的动画过程及图片文字显示等等。
5. **事件处理函数在`InitAllUIControllers()`中绑定，可是我还没见过什么事件处理函数咧**
6. 一个认识：这个框架之所以叫“框架”，并不是库的那种你拿过来随意组合调用的各种组件，而是给你规定了流程（*什么事情在什么时候做*），你要在这个流程中的固定位置填充你自己的逻辑。真的就是“别人调用你”
7. 加载的资源分为动态资源和静态资源：
   1. 静态资源就是每一个Task拥有的Layer
   2. 动态资源就是根据逻辑数据得到的不确定的资源
8. Task就是工作单，Manager就是执行者，Layer就是资源和材料
9. UITaskPipelineCtx好像就是保存一些本次Update时候的状态 
10. UICtrlDescArray记录了:
    1.  Controller绑到的layer的名称
    2.  绑定路径是什么
    3.  绑定的类型（就是那个Controller）是什么
    4.  类型的名称