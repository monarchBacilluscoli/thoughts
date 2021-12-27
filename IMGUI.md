# IMGUI

[来源](https://blog.unity.com/technology/going-deep-with-imgui-and-editor-customization)

## IMGUI和In Game GUI的区别：
   Retained GUI是保存了数值信息的（新建的slider你不用也可以拖拽并且显示值）。而IMGUI则不会保存，想想Slider的函数签名
   ```csharp
   public static float HorizontalSlider(float value, float leftValue, float rightValue, params GUILayoutOption[] options);
   ```
   它不仅会返回值也需要输入值，亦即每帧刷新的时候都会重绘并找你要值。

## Control ID
对于IMGUI来说，重绘之间的一些状态还是需要维持的（否则GUI.Button肯定没法运作，因为你没有传入任何它当前被按下的信息）。为了使之连续，相同的控件都持有了一个固定的ID和一组状态，他们是按照GUI控件函数被调用的顺序被分配的，所以保证事件间调用顺序一致非常重要。

## GUIUtility.HotControl
是HotControl设置的ControlID的Control，才能从Event.current.GetTypeForControl中拿到鼠标事件，非HotControl的鼠标事件则会被Filtered，返回Ignore

## GUIUtility.GetStateObject(Type, ControlID)
可以自定义一个状态对象给某个ControlID的控件，Unity帮你创建，每次绘制的时候只需调用这个拿出来就行    

## 如何复制并保存Unity原生GUIStyle
可以保存到ScriptableObject但是内建资产不可用。

##### 问题
1. 某字段会在OnGUI中根据鼠标位置进行设置，为什么我在OnGUI中判断该字段不空则绘制一个GUILayout control的时候会报错"ArgumentException: Getting control 0's position in a group with only 0 controls when doing Repaint Aborting"
   1. 因为Layout阶段在鼠标事件之前，鼠标事件中设置了字段，然后想去设置Layout类型的control的时候发现Layout阶段已经错过了，没有办法get到那个在Layout阶段尚不存在的组件的布局信息。
   2. 关于Layout和Repaint阶段:
   There are two passes to the OnGUI thread (they are called "events" in Unity), the layout event runs through and basically does roll call and layout planning on the GUI components and the repaint event draws the components to the screen. If the components change between these two GUI events, the layout manager gets confused and produces this message. The contents of your chat entries is externally controlled (from the OnGUI thread's perspective) and may be changing the contents of that collection between the events. This is forbidden.([参考](https://answers.unity.com/questions/17718/argumentexception-getting-control-2s-position-in-a.html))
   3. 解决之道：不在非Layout阶段进行可以改变下一次Layout阶段格局的数据修改操作
2. 在使用popupWindowContent的时候，每次修改代码都会有如下报错
   ```
   NullReferenceException: Object reference not set to an instance of an object
   UnityEditor.EditorWindow.Close () (at <007193b7fa9c4ad1be5b26df6a654213>:0)
   ```
   原来是编辑器的Layout没有保存，保存当前Layout，恢复default并恢复解决（[参考](https://forum.unity.com/threads/unity-editor-null-reference-exception.985474/)）
3. 一个让人非常想死的BUG：CDrawer持有了NodeInfo4Editor并对自己进行序列化来绘制这个Node;在单步调试经过EndWindows的时候这个值中的一些域如m_regionInfo被修改了;断点打在m_regionInfo的所有写入行都没有命中。   
   1. 经过consult发现是Unity在序列化（`new SerializedObject(this)`）的时候会将所有null的并且被`Serialized`的字段填充