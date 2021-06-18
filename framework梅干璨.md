# Launcher
GameManager.Initlize: Load GameSetting->SetResolution,targetFrameRate->LogManager->TaskManager->ResourceManager->AudioManager->SceneManager->UIManager【->Assembly->luaManager,热更方案已弃用】

GameEntryUITask: Start EntryPipeLine on PostUpdateView
EntryPipeLine: StreamingAssetsFilesProcessing->Load BundleData->CheckVersion->Download Bundle->Load BundleManifest->Load dynamic dll->Load lua->Load ConfigData->Start AudioManager->UITaskRegister->LaunchLogin

Load ConfigData: Load Assets->Deserialize from Assets->StringTableManager->ExtensionProcess(special)

# 配置数据
1. 通过配置excel表，实现类型定义和数据配置
2. 通过配置导出工具`ExcelConfigAnalysisTool.exe`，导出类型和proto数据
3. Unity中通过`BlackJack.Framwork.ConfigData.BuildAllConfigData`,自动生成脚本（通过模板插件NVelocity）和assets数据（将二进制文件变为ScriptableObject）
4. 加载时，先加载assets,然后反序列化，然后初始化StringTableManager,最后调用扩展流程（需要特殊处理的数据表）

# ResourceManager:
BuildAllAssets:ExportResaveFile2BundleAsset->GenerateBundleData->BuildAssetBundles->ExportBundleDataVersion->ExportStreamingAssetFiles

1. 开始StreamingAssets文件预处理（仅处理了resave文件）
3. 开始加载BundleData
4. 预下载bundle (m_touchedBundleSet)
	> 系统中曾经加载过的bundle的集合
   > 处于这个集合中的bundle将会被自动更新
   > 添加到这个集合的途径如下：
   > 1. streamingAssets预处理涉及的bundle全部加入
	> 2. 设置bundleData.m_isNeedPreUpdateByDefault 为true
	> 3. 游戏运行过程中曾经对该bundle发生过下载
	> 4. 调用函数AddTouchedBundle
	> 5. 调用函数StartDownloadUntouchedAssets
5. 加载BundleManifest

### 资源加载
LoadAsset
1. 尝试从缓存获取
2. 非Editor，或允许在Editor中加载bundle，调用LoadAssetFromBundleEx
3. 在editor环境下可以fallback到AssetDatabase.load
4. 最后fallback到resources.load
5. Cache
6. Prefab处理PrefabResourceContainer

LoadAssetFromBundleEx
1. 获取BundleData
2. 尝试从缓存获取
3. cache没有命中，调用LoadBundleEx
4. 从bundle加载资源

LoadBundleEx
1. 注册一个加载现场
2. 如果已在加载中，等待加载完成，否则进入下一步
3. 依次加载所有依赖的直接依赖
4. 加载bundle
5. Cache
6. 文件转存处理
7. 注销加载现场

### 资源缓存

AssetsBundle
>Bundle之间有依赖，进行引用计数
>根据引用计数和超时，进行定期卸载

Assets
>CacheItem弱引用持有资源，Reserve通过引用持有
>在关键点调用GC和UnloadUnusedResourceAll->Resources.UnloadUnusedAssets对未被引用的资源进行卸载

# Prefab
PrefabControllerBase
>所有prefab都应该有的脚本,直接控制UI元素，一般在代码中（UITask和UIController）进行创建
>AutoBindAttribute,通过反射将GameObject,Component绑定到脚本上

PrefabControllerCreater
>创建Controller到Prefab上，并绑定所有Field

PrefabResourceContainer
>继承于PrefabResourceContainerBase，未做任何修改
>实现嵌套的Prefab，Prefab在被ResouceManager加载成功后，获取所有Container并保证持有的资源被加载。
>load后进行Prefab的资源加载，在Controller中进行实例化


# SceneManager
主要管理三种类型的Layer(UI,3D,UnityScene)
维护了一个LayerStack,在Tick中进行排序和Layer所在层以进行3D层和UI层嵌套
层级关系主要由Push顺序确定
不同类型的layer通过修改camera的depth实现前后关系

# UIManager
主要对UITask进行操作，并持有所有UITask的注册信息，根据Group进行冲突分类，根据Tag统一操作一组UITask
StartUITaskWithPrepare配合PrepareForStartOrResume使用
StartUITask, StartUITaskWithPrepare, ReturnUITask
StartUITaskInternal: CloseAllConflictUITask->Add2Stack->StartOrResumeTask(Start,Resume,OnNewIntent)

### UIIntent
>UIIntent: TaskName & TargetMode
>UIIntentCustom: +m_params
>UIIntentReturnable: PrevTaskIntent (组合为一条UITask链)
>UIManager.IntentStack: 用于返回到链上更前面的Task

### UITaskBase
>一个功能的逻辑主体,可包含多个Layer，也可包含多个Task
>从PlayerContext获取数据，并传递给UIController以修改UI元素
>直接或间接(NetWorkTransactionTask)操作PlayerContext与服务器进行通信
>UpdatePipeLine: SetCurrentMode->UpdateDataCache->LoadStaticRes->LoadDynamicRes->UpdateView

### UIControllerBase
>继承于PrefabControllerBase，直接控制UI元素

# PlayerContext
PlayerContext: 所有玩家数据的整体，提供与服务器交互的接口，Request/Ack/Notification
LogicBlock: 对一个功能的客户端逻辑块
DataContainer: 数据集合和提供修改数据的方法
DataSection: 持有某一块实际数据，可进行本地序列化，以减少每次登陆时需要从服务器获取的数据量

# NetworkClient
PlayerContextBase(IPlayerContextNetworkEventHandler)->NetworkClient(IPlayerContextNetworkCLient)->Client(IClient)->Connection(ProtoProvider,Socket)
网络相关

# AudioManager
通过IAudioManager对声音解决方案进行封装
三种声音解决方案，Unity,WWise和CRI

# CommonUIStateController
UI通用状态机，通过Animator或Tweens实现UI动画，并控制GameObject的显示和颜色等
主要由美术使用制作prefab和各种状态，程序只需要调用state即可完成界面表现

# UIProcess
UI过程的基类，可以组织一组不同的Process，子Process可分并行和串行执行
主要用于在UITask中调用UI打开、关闭、切换等动画