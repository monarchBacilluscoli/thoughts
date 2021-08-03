# ClientFramework

## Task
客户端完成功能的集合
### UITask
负责更新管线，还有事件逻辑
#### 更新管线
逻辑数据获取，动态/静态资源的管理，界面刷新逻辑的控制
`UITask`管理若干个`UILayer`以及所属的`UIController`，负责创建并显示`Layer`，同时通过`UIController`控制`Layer`的动画过程及图片文字显示等等。

### NetworkTransactionTask
处理了服务器返回的几种错误情况，另外要在Task内部注册并取消注册处理返回数据的回调

## UIManager
图书馆、

## TaskManager


## UIIntent

### UIIntentBase
包含UITask的Name和mode
### UIIntentCustom
另含一个`Dictionary<string, object>`，可扩展的参数。这个参数的key（string）常常在要启动的Task本身之中定义。
### UIIntentReturnable
包含一个`previewTask`，即可以支持UITask之间的回退操作

## UITaskPipelineCtx
记录当前管线更新过程需求

## UIController
负责进行自动绑定、具体的UI行为定义、事件逻辑接口
事件处理函数在`InitAllUIControllers()`中绑定

## PlayerCtx
负责服务器事件处理、给外部提供端口对数据进行处理

## ResourceManager

### BundleData
包含了所有bundle的信息
### SingleBundleData
包含了单个Bundle的信息
### BundleDataHelper

### PrefabContainer
将依赖的prefab的实例创建在当前Prefab下
### ControllerCreator
将定义的Controller绑定到物体的对应位置
### 
