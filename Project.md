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
7. Build出错可能宏定义（Editor）。

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
12. 

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