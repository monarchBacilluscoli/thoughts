# 课程&实践
1. scene是文本文件
2. excel实际是二进制文件

## 要做的事情
- [x] Git上的各种文件夹分好
- [x] 程序组安排出组长
- [x] Jira里面整一套事情临时走一下过程.
- [x] 塔防作为Option

## What I thought & learned
1. 如果做水草的减速，对玩家减速和对`NavAgent`减速需要两套逻辑，怎么把这两套逻辑统一？
   1. 继承同一个基类？
   2. 做一些适配器？
   3. CTM，接口！
      1. 这时候接口的意义就显现出来了，没有数据和实现，没有构造器的菱形继承问题。干净！
   - [ ] 是时候看一下编程模式了
2. 状态机的大部分逻辑都是一致的，可以合并为一个基类或者一个组件
   1. 一个基类似乎并不合适
   2. 组件还不是要在Mono的各个函数中反复去调用那些方法？
   - [x] 查看合并的可能性
3. 音频和动画部分和逻辑要拆分，具体怎么拆分？
   1. `State`中的`SetGraphics()`?
   - [x] 理顺逻辑拆分
4. active和enable的区别
   1. `GameObject`可以被Activated，脚本可以被enabled
   2. 
5. 向上持有和向下持有应该如何做？
   1. 比如player有个hand作为投掷起点，手有自己的力度，但是设定时希望通过player设定，那么调用的时候应该是player设定hand的power还是hand不存power只从player中取

## What I can learn from other people
- gxy：
  - [ ] 后处理
  - [ ] 他所使用的TA工具
- K：
  - [ ] 建模所使用的工具
  - [ ] 如何搞定材质和贴图的

## What I should read
- 图形学：
  - [ ] 龙书
- 编程模式：
  - [ ] 游戏编程模式
  - [ ] Game Framework
- 计算机网络：
  - [ ] TCP/IP
- Unity
  - [ ] 重新捋一眼Manual（DR）

## 关于Debug
1. UI控件无法交互
   1. 可能是`EventSystem`没有挂
2. `Quaternion.LookRotation(Vector3 forward, Vector3 upwards = Vector3.up)`的用法：
   1. 将本地z轴和`forwad`对齐，将本地y轴和`upwards`对齐
   2. 这里实际上很有迷惑性，如果`forward`和`upward`并不是z和y的话，所以只需要记住，这个函数的作用是将z和y对齐即可。
3. 状态机的状态可以有一个`Initialize()`函数，当创建对象的时候用Mono的物体`Awake()`调用它.
4. 物体的动画逻辑跟组件关系很大
5. `UnityEvent`被修改之后需要重新登记
6. 动画修改了，一定记得Apply
7. Build出错可能因为宏定义（Editor、Debug之类）。
   1. 所以如果没捋清楚可以不用
   2. 要么所有的东西都包在一起
8. **重要！**Quaternion.FromToRotation()好像是差值，而LookAt()是绝对值
9. 两个Quaternion直接相乘是先用lhs后用rhs
10. `Quaternion.AngleAxis()`的意思是围绕某个Axis旋转多少度，而不是在这个方向上添加扰动旋转
11. OnLocalPlayerStart()只会在Client是Owner的时候调用一次，所以公有的挂载不要在这里写，而是在Awake或者Start中写。
    1.  此外，如果挂载的物品在RPC调用的时候为null，则null的客户端直接挂掉。
12. 粘墙bug：如果在空中时有向墙体冲的速度，那么人将吸在墙上不掉下来
    1.  妈的是物理材质的问题，默认的物理材质有**0.4**的摩擦系数
13. ScriptableObject作为了Buff的基类，希望在不继承MB的情况下可以图形化配置。但其中包含两类数据：
    1.  永久化数据：EffectTime
    2.  当前数据：CurrentTime
    3.  这样导致了问题：currentTime为一大于0的值，在构造函数中重置也没有用，并且将之Serialize显示出来后表明不仅在Runtime，在asset中也确实显示为该值。表明改值被序列化到asset中了
    4.  似乎`ScriptableObject`中的所有字段即使不加`SerializedField`也会被序列化
        1.  是的
14. 使用SO作为配置文件并拖放到脚本某字段时，在Editor中没有什么问题，但是Build时该字段变成了null，并有Serialization读取到比较少的数据的提示，并且四个config都少了12byte。
    1.  最终发现是SO中有`readonly`字段，导致这一问题。
    2.  原因：
        1.  Unity无法序列化`readonly`字段 https://docs.unity3d.com/Manual/script-Serialization.html
15. 所有的assets都可以用编辑器打开查看其中是否真正被序列化了东西。
16. 实际上自定义的类型是可以被显示在Inspector之中的
    1.  此外，如果挂载的物品在RPC调用的时候为null，则null的客户端直接挂掉
12. 关于SO null ref的问题，这个可能会有帮助：https://forum.unity.com/threads/my-scriptableobject-references-are-becoming-null-in-the-build.464206/
    1.  此外还有一篇关于Serialization的：https://blogs.unity3d.com/2012/10/25/unity-serialization/?_ga=2.244697421.998148984.1621437408-829177975.1618757098
    2.  对，没错，问题就是因为名称不匹配
13. 现象：`UpperBody`和`LowerBody`上对应的Mask后走路胳膊摆大字，没法让UpperBody在非Aim或者Shooting状态时跟随当前动画的手臂姿态。
    1.  原因很简单：LowerBody不能不勾胳膊，逻辑应该是LowerBody勾胳膊，但在Aiming或Shooting的时候用Upperbody的姿势覆盖原先姿势
14. 只有在SubStateMachine其中的状态有进入Exit的Transition时候，整个状态机才可能在不同的substatemachine之间转换
15. `LookRotation()`使用时请注意，以第一个forward为准，如果forward和up有冲突，则up完全没得效果，所以记得把forward的方向在up的plane上project一下
16. 网络开发的时候有一个模式，为了处理这类问题：
    1.  玩家进入一个网络物体的碰撞体，捡取了该物体——然后向服务器发送摧毁该物体的请求——服务器摧毁了但客户端还没摧毁——客户端再次发生碰撞——捡取/摧毁了两次

## 关于Mirror
1. 设置玩家
   1. 给到可以同步的`SyncVar`
   2. 使用`Command`来设定client->server对这些玩意的设置。
   3. 给这些`SyncVar`挂上hook，让他们被改变的时候可以自动设置并非SyncVar的参数的值（因为SyncVar只能是简单类型，复杂类型无法直接被Sync）
   这样就可以以一种Command的形式调用server scene的玩家的修改了——暴露出来的修改Command，Sync的Var以及hook构成了不同客户端交互的手段。
2. 这里似乎暗含了几个限制
   1. server到client和client到server的逻辑要通过attribute分开
   2. 服务器会调command嘛？
      1. 不会
      2. 断点被命中的原因是因为**Host = client+server**
         1. 当前玩家（Editor）开启Host的时候会命中
         2. Client玩家开启Client的时候也会命中
3. `SyncVar`是
   1. 由服务器自动同步（无需管理）给客户端的
   2. 继承了`NetworkBehaviour`的类中的properties
   3. 所以修改他们的值需要由server来进行，从clinet上修改是pointless的
      1. 于是修改这些值的方法要写`[Command]`
4. 它的修改逻辑是使用local值作为临时值，计算完毕后修改SyncVar...莫非在一个Update中的多次修改也会引发大量网络调用？
   1. 或许读取的时候也会存在逻辑？
5. `Command`是
   1. Commands are sent from **player objects on the client** to **player objects on the server**. 
   2. Commands can only be sent from **YOUR player** object by default
      1. 所以所有的同步请求都要从`Player`去的话，所有的物件都必须要持有`Player`了
6. 带有`NetworkIdentity`组件的对象会在开局但是Player并没有Spawn的时候被disabled。
   1. 所以如果需要在开局的时候持有这些组件，也许需要将这些组件挂在另一些不被disabled（非网络）的物体上
7. 创建网络物件，需要用`NetworkServer.Spawn`才行，这样创建的物件会有一个`netId`，才会被服务器同步到所有客户端中
8. `isServer` —— True if this object is on the server and has been spawned.
9. networkmanager中的offlineScene使得掉线玩家会回到这个，并且访客会在房主掉线时回到这个
10. 客户端 actually shares the Scene with the server.
11. `Network Object`的作用是同步该物体的创造和消除。而别的东西则不会管理
12. 怪不得Network GameObjects需要注册才能在客户端创建，就像两边都需要数据集才能一边序列化一边反序列化一样，网络传输靠得也是序列化。所以需要数据集

### 问题
2. 玩家的状态都可以在服务器上修改并且同步到客户端，根据hook可以对客户端的表现进行修改
   - [ ] 但是创建物体呢？
- [x] 我把碰撞并调用修改player血量的逻辑写到本地的bullet的OnCollision()里了，按道理说会执行客户端数量的次数，为什么没有？
   1. 首先，执行了客户端数量的次数的command call
   2. 但是，command call调用的是player.ModifyHealth()，这个是否成功，要看client有没有这个player的authority，显然只有一个client有，所以抛出了一个warning，并执行成功了1次。
- [ ] 我可以修改player的数据了，但是非Player的数据呢？
  - [x] 比如我创建了一些物品掉落，是可以在每个client中分发
  - [ ] 如何捡起该物品？
    - [ ] 看一下教程
    - [ ] 据说responable的物品要在Manager中registered，为什么呢？
      - [ ] In addition to the Player Prefab, you **must also** register other prefabs **that you want to dynamically spawn during game play** with the Network Manager.
  - [ ] 如何不捡起该物品但是可以控制该物品？
- [ ] 不是`Monobehaviour`如何才能在外部配置它的属性？
- [ ] 如何合理利用继承？
  - [ ] 例如父类具有一个计时和最大时间的字段，所有子类都需要同一套计时逻辑在`Update()`中实现，但是`Update()`中按道理说还需要进行具体的行为应用，这里的最佳实践是什么？
    - [ ] 父类如果把`Update()`函数设置为`virtual`给子类，那么这里没有手段要求子类必须要首先调用父类的`Update()`
    - [ ] 父类如果真的不实现子类，那么把`Update()`给子类实现子类需要查看父类的字段来揣摩意义
    - [ ] 父类实现了`Update()`，然后在Update中调用子类的`UpdateApplyLogic()`，这样很多叠加嘞
    - [ ] 该问题以及相关的解答体现了怎样的哲学？
- [ ] 玩家拾取物品的逻辑应该如何做
  - [ ] 尤其是加入了网络部分


### 游戏2的整体对局逻辑
1. 玩家加入，分配队伍
  2. 此时不开始计时
  3. 同一队的玩家分配在一起（应该有四或六个出生点）
2. 开始对战，开始计时，记录占领地块
  3. 玩家死亡逻辑处理
    4. 倒计时，复活位置
  5. 占领逻辑处理
    1. 占领地块
    2. 怪物刷新
    3. 道具掉落
3. 时间结束，统计双方占领数量，数量多者获胜
4. 按下按钮，游戏重新进入起始状态

又是状态机：
准备（添加玩家、时间暂停、房主可以决定开始时间？）——进行（不允许加入（当然可能也挡不住），执行游戏更新逻辑）——结束UI（按钮重新计分）

此外还要处理的内容：
1. UI
  2. 血量
  3. Buff及时间（optional）
  4. 游戏启动
     1. 开局标题+进入游戏
     2. 开房间/客户端加入房间（坐一起算了）
  5. 游戏结束
     1. 计分页面
2. 道具外观
  6. 加速器
  7. 血包
  8. 速射枪机
3. 音频
  4. 射击
  5. 人物移动
  6. 道具获取
  7. 占领
  8. opening title背景音乐
  9. 胜利音乐

### 游戏2第二次开发选型
#### 网络：选择Mirror的原因
1. 在第一阶段的开发当中，Mirror已经被证明可以很好地满足开发的需求：
   1. Mirror相对于自行构建框架的优势
      1. 开箱即用的tansform、rigidbody、Animator等component支持
      2. 在保持逻辑完整的情况下改变方法特性即可将逻辑置于客户端/服务器
      3. 嵌入式支持，和Unity无缝衔接，减少服务器和客户端的开发/沟通成本
      4. 稳定
      5. 简单浏览doc后每个人都可以处理服务器同步逻辑，而不是仅由一人来维护/修改服务器逻辑
   2. Mirror在重构代码之下的转向（如果需要单独抽出server逻辑）
      1. 将服务器逻辑单独抽出即可
      2. 利用底层接口支持（需要查看doc）
    1. 和第二阶段目标相关：
      3. 这次目的恐怕并非是练手，而是上线，这里时间也并不充裕，需要一个比较耐得住颠扑的基础设施
### double二次开发新特性技术参考
1. 道具
* we'd have a **GameObject** reference where the *attachment point* can be assigned in the inspector, a **SyncVar enum** with various choices of *what the player is holding*, and and a **Hook** for the SyncVar to *swap out the art of the held item* based on the new value.
* 由于这里不提供调节子物体相对位置的方案，所以可以 having the grip or handle of an item align with the hand, 从而根据这个来调节子物体的相对位置
* Remember that, as child art, the item prefabs *have no networking components* on them at all.
  * 用于处理掉落逻辑的时候，貌似需要特别处理
  * 即需要另一个和玩家切枪一样逻辑的item container，这个container也可以像玩家一样接受一个enum然后切换实际包含的道具

2. 问题：
  1. **确认该框架可以用于商业游戏开发**
#### Git的开发模式
取决于个人习惯：
1. 使用个人分支——常常`commit`，每个`Feature`完成时`merge`到`develop`
2. 
