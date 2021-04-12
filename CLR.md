# CLR via C

1. p15页提到IL没有提供寄存器指令，所以比较容易创建新的语言和编辑器。
这一句的意思应该是将基于寄存器语言和基于栈的语言进行对比，栈的操作相对来说比较简单，可能主要就出栈入栈top之类，而基于寄存器可能要考虑各种平台的寄存器差异，寄存器数量也比较多所以可能需要定义的操作啊啥的比较多，于是说基于栈的IL可能比较好开发新的语言。

2. 清单元数据表听起来好像头文件一样啊

2. !注意！c#仅仅允许将对象转换为它的实际类型或者它的基类型，而不能是什么别的类型。

3. 程序集和托管模块，清单和元数据

4. 什么是PE文件

5. 项目、程序集、模块
	为什么要有模块这一独立单元？

6. 实用工具:
	* AL.exe 程序集链接器 - al
	* csc 估计是csharp编译器
	* ILDasm.exe 反汇编程序 - monodis
	* NGen.exe - monogen
	* SN 强命名（Strong Name）密钥生成程序 - sn
	*  - 运行时 mono -debug	
	* f#的编译器 - fsharpc
	* PEVerfify
	在mac上以上的带有"-"符号的都可以用，并且语法一样


7. RSA（非对称加密）的加密解密和签名的原理
	首先要知道对称加密，就是密码本发给对方，我用这个加密，你用反方法解密。
	好，这样一来密码本就很危险，一旦被截获就毫无秘密可言。
	于是乎诞生了RSA，即加密和解密的方法不是方法-反方法的关系，而是毫无关系。	也就是说加密可以用一种方法-公钥，解密可以用另一种方法-私钥。
	作为数据的接收者，你只需要传给发送者自己的公钥，然后发送者使用之加密，你使用自己的私钥解密即可。私钥是在你手里永远不会传到网上的，也就是说没有私钥，除了发送者知道消息是什么之外，别人没法知道。
	但是这样的场景存在一个问题，就是你无法知道信息是不是真正来自你要的发送者，它可能是来自其他人的假消息。
	这样一来RSA的反向用法-签名就派上用场了：
	1. 用于验证发送人
	2. 从而用于防止篡改
	就是用发送人的私钥对消息进行加签，然后用发送人的公钥对消息进行验签，既然没人知道发送人的私钥，就没人能模仿发送人给你发消息,耶。

8. C#的托管堆进程栈和对象以及对象类型对象这一套东西真几把爽快，逻辑链很通畅啊！
	两条线：一条是Object，一条是Type。
	Object是明线，无论什么类型都继承自它。
	Type类型对象是暗线，每个Object之中都有这么一个“隐藏对象指针”，指向这个类型对象。
	一个是继承关系，一个是包含关系。
	Type类型对象存储的是类数据，Object对象存储的是对象数据。

9. 同步索引块是什么玩意？

10. 值类型的方法存在哪里，值类型变量如何存储这些方法的入口？每个值类型都有所有的方法入口？还是每个值类型也有一个指针指向一个类型对象实例？

11. 装箱和拆箱存在的意义是什么？跟c#的强类型逻辑有关系吗？难道是为了性能先有了值类型和引用类型的区分，然后发现这两种类型之间交互会存在一些问题，于是诞生了这个反过来看起来严重影响性能的玩意？那么这里的权衡又是什么呢？
那么看第12点，我的理解确实是上面这个说法。

12. 装箱和拆箱的场景是什么呢？
	1. 调用一个含Object参数的方法，这个方法可以通用，但是对于值类型，需要装箱。
	2. 非泛型容器，需要装箱（第1点是它的具体逻辑）

13. ref和copy的传参有何区别？
一般的copy传参，无论怎样都会copy**栈上**的对象：
	* 如果传入的是指向obj的变量，那么copy之，造成两个后果：
		* copy的和origin的都指向同一个对象，利用copy的修改，origin的也会改变。
		* copy的和origin的是两个变量，copy的改变指向不影响origin的。
	* 如果传入的是valuetype变量，那么copy之，造成一个后果：
		* 你不管怎么修改copy的，origin的都不会变。
而ref的则不同，ref的形参就相当于实参的别名，造成一个后果：
	* 无论你使用形参调用实例方法进行修改还是直接改变形参指向（实参是引用类型变量），这个实参都会跟着变。
他们两者在IL代码上有啥不同呢？
比如我们看一个简单的Swap俩object的函数：
```csharp
static void Swap(ref Object o1, ref Object o2)
        {
           Console.WriteLine(o1.GetType());
            Object t = o1;
            o1 = o2;
            o2 = t;
        }
```
它的IL代码就是这样（和非ref的差别用**//**标出）：
.method public static hidebysig
           default void Swap (object& ro1, object& ro2)  cil managed
    {
        // Method begins at RVA 0x2068
	// Code size 12 (0xc)
	.maxstack 2
	.locals init (
		object	V_0)
	IL_0000:  nop
	IL_0001:  ldarg.0
	IL_0002:  ldind.ref	//
	IL_0003:  stloc.0
	IL_0004:  ldarg.0	//
	IL_0005:  ldarg.1	//
	IL_0006:  ldind.ref	//
	IL_0007:  stind.ref	//
	IL_0008:  ldarg.1
	IL_0009:  ldloc.0
	IL_000a:  stind.ref	//
	IL_000b:  ret
    } // end of method RefAndValue::Swap
最显著的差别在于：
	* ldind.ref，吃一个栈顶的变量（地址），得到这个变量指向的对象的引用，吐一个这个参数指向的对象的引用到栈顶。
	* stind.ref，吃两个栈顶的变量，把第一个变量（值）存储到第二个变量（地址）指向的位置。
最后这俩可能只交换了两个实参引用的指向。（待验证：是的没错）
还有一点补充，就是调用ref参数函数之前，在外部栈帧构建实参的时候，ref是用的ldloca，非ref是用的ldloc

https://www.cnblogs.com/murongxiaopifu/p/6616901.html
是啊，实际上c#使用这个语法糖是想让我们把引用的形参看作是实参的别名，也就是和直接在外部调用实参是一样的，如果这里交换的是实际指向的内存里的数据，那么在外面直接调用的时候也应该是，于是就会出现这样的情况：
Object a = new Object();
Object b = new Object();
a = b; 
这样a和b就不是指向一个对象了，而是b把指向的内容拷贝给了a，如果这种说法不成立，那么肯定就不是这样：
显然不是。

14. c#之中能不能获得指针嘛？
好像值类型可以，引用类型有特别的办法。

15. 引用类型还是会踩坑。
这就是前面需求的动机。

16. LINQ是啥

17. 为啥set函数之中的value在IL之中是Int32 'value'? 为啥有这个引号？

18. arg.0 是啥
在成员函数之中，应该是this。

19. 一些IL指令以及其效果
ldfld: 吃一个对象ref（pointer），吐一个该成员的value出来。
ldc.i4.0: 应该是load const interger 4 bytes which value is 0，也就是将一个4的const load到栈上。
newarr EventTest.Luckyfriend：吃一个数组大小（int），吐一个对应大小的指向一组Luckyfriend的数组变量到栈上。
一些跳转语句：
br.s IL0023 : 将控制无条件地branch到label IL0023语句上
brfalse.s：一旦是false（应该是stack上最后的变量是false），branch到后面的label：
clt：吃一个value2，再吃一个value1，如果1小于2则返回1.
stelem: 应该是吃一个value，然后吃一个index，然后再吃一个array，最后将value存储到array[index]中
ldloca index: 吃一个local variable的index，然后将对应的var的地址吐到stack上。值类型成员函数调用的时候经常用，毕竟要把值类型的this指针传过去。而引用类型的函数调用用不着，直接ldloc就行，毕竟引用类型变量存储的就是地址。但是注意，ref函数调用之前，ld的就是这个loca，京哈哈哈哈啊解决了这一问题哈哈哈哈。
ldftn
newarr 吃一个size，创建size大小的newarr后面参数类型的数组并将引用push到stack上
20. 说了这么多，啥是类型安全
21. 看到了一些Interlocked方法, interlocked提供了一些原子操作给用户
Interlocked.Exchange() - set a variable to a specified value as an atomic operation
CompareExchange(ref a, b, c) 比较a和c的值，如果相等，则更新a为b。换个说法，看a还是不是c（一般先把a copy出来的一个值），如果是，那就说明在a加到c的过程中a没改变，不存在其他线程竞争，那就安全地把想更改到的值b放到a中。

22. 关于volatile.read
	1. 关于内存栅栏
	添加一个memory barrier会向编辑器告知“执行当前线程的处理器不能将这个栅栏之后的内容提前到这个栅栏之前执行”。防止编辑器瞎调语句执行顺序。
	2. 关于volatile关键字，MSDN中有一个描述很有意思，所有的ref、pointer、简单类型、枚举都可以被修饰，但double这种就不行，because reads and writes to fields of those types cannot be guaranteed to be **atomic**. 这也就是说读取前面这些类型都是atomic的啦？看来只要加了volatile.read就是线程安全且真实的了。
	3.原子性+指令顺序保证=多线程安全?

23. 托管和非托管
　为CLR而编写以及使用CLR服务的代码叫"托管代码",而那些未使用CLR服务的代码(也就是你多年以来一直编写的代码)叫"非托管代码".
托管对象指的是.net可以自动进行回收的资源，主要是指托管对象在堆上分配的内存资源。 托管资源的回收工作是不需要人工干预的，有.net运行库在合适的时间进行回收。

24. 什么是BOM
23. 托管和非托管
　为CLR而编写以及使用CLR服务的代码叫"托管代码",而那些未使用CLR服务的代码(也就是你多年以来一直编写的代码)叫"非托管代码".
托管对象指的是.net可以自动进行回收的资源，主要是指托管对象在堆上分配的内存资源。 托管资源的回收工作是不需要人工干预的，有.net运行库在合适的时间进行回收。

24. 什么是BOM
23. 托管和非托管
　为CLR而编写以及使用CLR服务的代码叫"托管代码",而那些未使用CLR服务的代码(也就是你多年以来一直编写的代码)叫"非托管代码".
托管对象指的是.net可以自动进行回收的资源，主要是指托管对象在堆上分配的内存资源。 托管资源的回收工作是不需要人工干预的，有.net运行库在合适的时间进行回收。

24. 什么是BOM
BOM是在Unicode编码的文件最前方出现的非显示字符，a magic number，用于标识：
	1. endian，字节序是大端还是小端。
	2. 到底是utf8还是utf16
	3. 是不是unicode编码
但是1.Unix全面拥抱utf，不用区分。2.和Unix的“文本文档里所有的字符都要显示“的哲学不一致，于是Unix不怎么鸟这个。于是就成了一个相对比较糟糕的设计。

25. Unicode是怎么回事
	1. ASCII不足以表示所有欧洲国家的本土语言字母，产生了扩展的ASCII
	2. 扩展的ASCII在不同国家表示不同的字符（128-255），尤其是象形文字，汉字就10万个，标准不一致+contain不了，需要新标准
	3. Unicode横空出世，设定了一个符号集，把整数和符号一一对应，Unicode 目前规划的总空间是17个平面（平面0至16），0x0000 至 0x10FFFF。每个平面有 65536 个码点。即可以有16*65535个字符。
	4. 但是Unicode只是一个”符号集“，规定了二进制代码，但没有规定应该如何存储。如果我用4个字节存储Unicode表示的ASCII，那么很浪费，但是如果我用变长的编码，那到底几个字节是一个字呢？如何统一呢？
	5. 于是出现了多种unicode的存储方式
26. UTF8是怎么回事
	首先这玩意是变长的(!)，其次用先导位来标注字符间隔。
	* 一个字节的编码和ASCII一样，第一位为0，后面是编码
	* 大于一个字节的，n字节编码，字符表示的前导位是n个1再加一个0，然后每一字节的前导位都是10.
	* 这样程序在check字符的时候根据前导位就知道用几个字节表示，并且假如中间断了也知道是断了，而不是单个字节表示的字符。

27. value type和ref type的 method call的差异，以及没有class pointer它怎么调用。
https://cis300.cs.ksu.edu/appendix/syntax/reference-value/
https://stackoverflow.com/questions/436363/does-calling-a-method-on-a-value-type-result-in-boxing-in-net
什么时候发生装箱。
我大体知道了，如果这个value type重写了继承的函数，或者自有的新函数，调用这些方法的时候不会装箱，函数获取的参数应该也会由CLR处理为value type，直接吃栈上的value。
但是如果需要利用继承，也就是用value type的基类变量调用虚方法，或者方法需要传一个基类的变量（实际上这俩是一个意思，毕竟成员函数是隐式有this指针），那么这个调用就要使这个value type发生装箱。
那么拆箱呢？我用一个在堆中的value type调用成员，比如数组中的value type，那里存的不是包装的value type啊，那个就是value type的裸数据啊，这算是需要unbox吗？毕竟我在IL中看到的也是ldelem，这个应该是直接ld了value的裸数据上栈了，这不算拆箱吧。
看看拆箱的定义。
>Unboxing is an explicit conversion from the type object to a value type or from an interface type to a value type that implements the interface. An unboxing operation consists of:
所以，就这个定义来说，从array中拿来value，看起来可能不需要经过unbox这一步特别的操作。虽然说起来array[index]实际上也是一种“引用”到stack上的值的操作。

28. 为什么理论上Timer在Debug模式中不会被GC呢？
因为VS做了小手脚，如果我们在调试的过程中想看某个变量的值的时候那个变量由于过了最后一个引用的点而被GC了那岂不是很握草。

29. class修饰的internal和class内部的方法public的差别。

30. .NetCore不能在Mac上打开第二个AppDomain
>The point of the .NETCore subset was to keep a .NET install small. And easy to port. Which is why you can, say, run a Silverlight app on both Windows and OSX and not wait very long when you visit the web page. Downloading and installing the complete runtime and framework takes a handful of seconds, give or take.

>Keeping it small inevitably requires features to be cut. Remoting was very high on that list, it is quite expensive. Otherwise well hidden, but you can for example see that delegates no longer have a functional BeginInvoke() method. Which put AppDomain on the cut list as well, you can't run code in an app domain without remoting support. So this is entirely by design.
from https://stackoverflow.com/questions/27266907/no-appdomains-in-net-core-why

31. 封送的对象如果含有引用类型的字段怎么整。按引用封送的就按引用整呗，那要是字段没有按引用封送的特性呢？按值封送的呢？
按值封送带reference对象的解答很显然在序列化这一章。就是字段到底是怎么被序列化的
至于按引用封送带引用的那么答案已经很明确了，每个字段都是代理。

32. 如果在按值封送的对象上执行带ref param的实例方法了？
用一个ref的数组查看吧。可以被fixed的，可以看地址的。
猜测它应该跟返回值的逻辑一致，都需要满足按值封送的条件，通过按值封送来传递。

33. 为什么GCHandle不能wrap class，但是可以数组嘞？

34. 吴悠给提供了一个这样的网站，和算法有关的，上去就是A*
https://arongranberg.com/astar/docs/getstarted2.html
不是上面这个，下面这个才是啊哈。
https://www.acoj.com/

35. 如何修改cdproj允许unsafe代码
```
<PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp3.1</TargetFramework>
    <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
</PropertyGroup>
```
就是AllowUnsafeBlocks那一句

36. 并不是所有string值一样的都会被“字符串留用”啊，默认只有字面值是字符串留用的。比如你用Console.ReadLine的就不会自动被留用，否则每次创建对象还要进哈希表，反而是费事了。

37. 除了字符串留用这种相对runtime的技术，还有一种compalier的技术就是字符串池——他妈的字符串池就是字面值字符串留用。跟前面那个一毛一样应该。

38. IFormatProvider是什么
用来处理语言文化的设施。

39. public Int32 Data => m_data>100 ? 100 : m_data;

40. 如果派生类用new重写了基类的接口方法，那么基类变量转型到这个接口类型调用这个方法实际会调用哪个方法呢？
当然是派生类的，转型到接口就无涉这个变量类型了，是对象的实际类型和接口的关系，和变量就没关系了。

41. 接口和接口之间的继承符合继承模型吗？
符合，父类接口没有实现的方法不需要子类接口实现。而类对接口继承则必须实现未实现的接口方法。

42. windows下C++编译器可以同时写native代码和managed代码？使用/CLR即可，可以试试。

43. GC的所有细节机制：大对象，finalize以及别的

44. 句柄到底是啥玩意，咋啥都需要句柄，文件。mutex咋都需要。

45. Action闭包的实现机制
同一个路径下的IL代码文件有介绍。
其实一开始疑惑有一个：
1. lambda表达式是按值捕获还是按引用捕获。
然后代码跑了一下发现是按引用捕获。然后问题就来了，如何按引用捕获？难道CLR为Lambda表达式构建的类型里面有外部变量的地址指针？那样的话：
	1. 如果栈展开啥的情况下，外部的局部变量的生存期咋保证呢？
	2. 为了实现这个ref有什么iL的复杂的特定语句？
然后就看了看IL，发现害有点精妙：
最直接的针对前一个问题的答案就是，这里没有什么指向外部变量的指针。没有指针，那lambda内部字段怎么和外部变量联系的?
首先我们知道lambda表达式定义时实际是定义了一个带有所谓的捕获对象和成员函数为所定义表达式的匿名对象，而外部函数里的局部变量实际上是嵌套类的字段！对外部变量进行任何操作都是对局部变量进行操作！	
这也就解释了为什么在程序一开始，所有的lambda中出现的具名外部变量都没有在.locals init块中出现and函数最先做的是构建lambda表达式的对象——因为所有的外部具名变量都是这个对象里面的字段，所有对外部具名变量的操作都依赖这个lambda对象
这一点也通过在程序外部调用i++得到验证。
好，那么如果你确实想实现C++之中的按值捕获怎么办？
很简单，想办法让lambda对象再保存一个状态即可，即在lambda外部用别的对象copy一下目标对象的当前值，然后定义Lambda的时候使用这个对象。那样CLR就会帮你安排另一个字段存储这个临时变量了。

46. 阅读下列代码：1、顺序写出控制台打印日志，2、修改Derived3使其实现IDisposable接口，并能满足Derived3的子类能重写IDisposable声明方法。
internal Interface IDisposable {
       void Dispose();
}
internal class Base:IDisposable
    {
        public void Dispose()
        {
            Console.WriteLine("Base");
        }
 
} 
internal class Derived : Base, IDisposable
{
   public void Dispose() {
        Console.WriteLine("Derived");
    }
}
internal class Derived2 : Base // 注意，继承自Base
{
    public new void Dispose() {
        Console.WriteLine("Derived2");
    }
}
internal class Derived3 : Derived
{
}
 
static void Main(string[] args)
{
Derived d = new Derived();
d.Dispose(); // Derived.
Base b = d;
b.Dispose(); // Base.
IDisposable id = b;
id.Dispose(); // Derived
Derived2 d2 = new Derived2();
d2.Dispose(); // Derived2
Base b2 = d2;
b2.Dispose(); // Base
IDisposable id2 = d2;
id2.Dispose(); //? Base
Derived3 d3 = new Derived3();
d3.Dispose(); //? Derived, 上溯吧
Base b3 = d3;
b3.Dispose(); // Base
IDisposable id3 = d3;
id3.Dispose(); // Derived
}

	几点需要澄清的知识：
	1. callvirt: The method metadata token provides the name, class and signature of the method to call. The class associated with obj is the class of which it is an instance. If the class defines a *non-static* method that matches the indicated method name and signature, this method is called. Otherwise *all classes in the base class chain of this class are checked in order*. It is an error if no method is found.
		1. How to check in order? From derived to base?
	2. The metadata token carries sufficient information to determine whether the call is to a static method, an instance method, a virtual method, or a global function. In all of these cases the *destination address is determined entirely from the method descriptor (contrast this with the Callvirt instruction for calling virtual methods, where the destination address also depends upon the runtime type of the instance reference pushed before the Callvirt*).
		1. How to call... of course it can call any method, directly decided by the token. 
		2. It is *valid to call a virtual method* using call (rather than callvirt); this indicates that the method is to be *resolved using the class specified by method rather than as specified dynamically from the object being invoked*.
	3. If I remember correctly call doesn't check the pointer against null before performing the call, something which callvirt obviously needs to. That is why callvirt sometimes is emitted by the compiler even though calling non-virtual method.
	4. 重量级知识：方法表及方法调用(https://www.cnblogs.com/OneRipple/p/7093876.html)
		1.首先，方法表中具有所有方法按顺序排列为：继承的虚方法（无论是否实现），新建的虚方法，构造函数，静态方法，实例方法。（点出顺序是为了表明继承是多态的基础，因为继承的虚方法无论怎样都是在同一个位置）注意，这里面没有继承基类的非虚方法。
		2.方法的调用：
		在调用方法的时候，C#编译器关注的是引用变量的类型，它并不会关心实例类型是什么。C#编译器会从引用变量的类型开始向其父类逐层查找：
			1. 从C#到IL：
				1. 对于虚方法，上查到出现virtual的那个方法签名，callvirt之。
				2. 对于非虚方法，上查到第一次出现该定义的时候，call/callvirt之。
			2. 从IL到Native Code
				那就只是call和callvirt的差别了，都好说好说。
	还有问题：new、interface的 virtual sealed会对方法表带来什么影响？

47. CLR中的继承在内存里是如何表示的？子类的类型对象会有指针指向父类的类型对象吗？
不会，子类会在创造的时候copy一份父类的数据，如果存在override则会override之。然后两者应该就毫无关系了。

48. 对于值类型，也有方法表（任何类型都有方法表），但值类型的实例没有类型对象指针指向它，需要访问类型元数据获得方法表。
http://c.biancheng.net/view/3093.html

49. 散列表相关概念：
	1. 开散列（线性探测，二次探测，菲薄拿起探测）
	2. 闭散列（这种称为哈希桶）（拉链法）
	3. 负载因子
	散列表的原理：妈的哈希表里面用的是key的hash_value！跟Value的HashValue没啥关系，草了！

50. 阿瑞推荐看算法的时候考虑先看A*，和地图的烘焙

51. 浮点数的机器表示。
https://www.ruanyifeng.com/blog/2010/06/ieee_floating-point_representation.html
符号位+指数位+有效数字位(有效数字位>=1且<2)。

52. Remoting、AppDomain以及Code asscess security在除.netframework中不被支持。
53. Error: Your project does not reference ".NETFramework,Version=v4.6.1" framework. Add a reference to ".NETFramework,Version=v4.6.1" in the "TargetFrameworks" property of your project file and then re-run NuGet restore.
	在改csproj从而使得.net 5.0变成.net framework的时候出错了，方法很简单，删除obj文件夹
54. 在出现/d /c的错误的时候，将task.json或者launch.json的出错的项目里加入`{"executable": "powershell.exe"}`


# 复习部分
## CLR的执行模型
### 将源代码编译成托管模块
1. CLR只是一个运行时，不包含编译器。可以简单视作是运行IL代码的环境。
2. 托管模块的组成
   1. PE32和PE32+头
   2. **CLR头**：这里有CLR的版本，**入口**，**签名**并且是**一级元数据表**实际上，因为它有真正的元数据表的大小和偏移量（CLR运行需要知道的内容）
   3. 元数据
   4. IL
3. 代码验证是什么
### 将托管模块合并成程序集
1. 程序集：含有清单的托管模块（这就是区别了）
2. 清单：代码文件、**公开导出的类型**及资源（似乎不包含def和ref元数据的类别）+ **程序集描述**（名称、版本、文化、公玥）
3. 程序集逻辑表示vs物理表示

### 加载公共语言运行时
1. /platform开关决定32还是64
2. 位数在PE头中，决定了调用的MSCorEE.exe版本
### 执行程序集代码
1. JIT：将IL转换为本机指令
2. 运行流程：
   1. 碰上Main()，将其中所有的`Type`构建出来（估计要查看这些`Type`的元数据）
   2. 所有方法指针指向JITCompiler
   3. 执行到函数时JITC**从`Type`元数据**拿到IL地址并：
      1. **验证**
      2. 编译成本机指令
   4. 修改方法指针
   5. 下一次直接执行native code
3. `/optimize`和`/debug`
   1. `/optimize-`将添加NOP和跳转到下一句指令（提供“编辑并继续”以及“跳转语句上的断点”）
   2. `/debug:full`记录源代码->IL->本地代码的映射，允许调试器单步进行。
   3. `NGen`直接生成本地代码
4. <span id="nativeveri">>验证</span>
   1. 在IL->native的时候
   2. 验证内容：
      1. 参数数量正确、类型正确、返回语句正确
   3. 因为IL是无类型的，所以才会需要在编译之前的多出一步
5. NGen的作用：（相当于预加载、烘焙等等）
   1. 提高启动速度
   2. **共享内存**减小程序集
   注意：它仍然需要元数据for**反射、运行失败**等
### 通用类型系统
1. **通用类型系统CTS**：相同并公开的类型结构允许不同语言之间进行交互。（相当于协议）
2. 公共语言基础结构CLI：包含CTS和文件格式、元数据、IL等其他组件的标准
3. 类型和成员的访问修饰规则：
   1. 类型：`public`和`assembly`（*顶级类型，而嵌套类型则视作成员*）
   2. 成员：`private`、`family(protected)`、`public`、`assembly(internal)`——包含所有的标准c++修饰符和程序集修饰符
4. **公共语言规范CLS**：所有语言中都支持的功能
   1. 在**导出且需要别的语言合作的类型**中需要遵守
   2. 可以使用attribute来进行CLRCompliant检查
   3. 这个东西随处可见，比如IL中仅存字段和方法（将其他的一些东西在编译器中转换成他们了）

## 生成、打包、部署和管理应用程序及类型
### .NET Framework部署目标
1. 响应文件：包含一组**编译器命令行开关**的文本文件
### 将模块合并成程序集
1. 为什么要引入程序集？
   1. 因为这样可以把物理实现和逻辑表示分开
2. CodeBase元素的作用
3. 清单元数据表的内容：
   1. **程序集描述**：**名字、版本、culture、公钥**（决定了强命名程序集。CLR头中包含了签名），这些东西显然不应该是CLR头里有的，比如CLR应该culture中立、culture可能也会影响公钥啥的
   2. 文件
   3. 资源？（和文件协同处理/link资源或单独处理内嵌(PE)资源）
   4. 导出类型
4. 编译器要求所有元数据实际就是所有文件都在，但运行时并不要求，只要没碰到就没事。
5. 程序集链接器并不修改或合并已有的module的内容，只是创建一个新的描述/入口文件。

### 简单应用程序部署
1. 什么是**私有部署的程序集**
   1. 应用程序基目录或子目录部署
   2. 不会被其他应用程序共享
### 简单管理配置
1. 如果某些依赖的私有部署的程序集文件没有部署到和主调文件同一路径会有**FileNotFoundException**异常
   1. 可以配置一个`程序名.exe.config`文件在基目录，其中加入privatePath把被调文件的路径写上，CLR就会自动去那里搜索

## 共享程序集和强命名程序集
### 两种程序集，两种部署
1. 强命名程序集：使用发布者密钥进行了签名的程序集
2. 弱命名程序集和强命名程序集部署方式的区别：弱命名只能私有部署
3. 强命名程序集的四个重要特性：
   1. 文件名
   2. 版本号
   3. 文化
   4. 公钥
### 为程序集分配强名称
1. 过程：
   1. 创建公钥和私钥
   2. /keyfile:密钥文件（只能用于含清单的程序集文件）
2. 内部机制：
   1. 第一次哈希：程序中所有文件都哈希存到清单FileDef中
   2. 第二次哈希：对PE文件进行哈希
   3. 对哈希值用私钥进行签名嵌入CLR头中
   4. 公钥嵌入元数据AssemblyDef中
3. AssemblyRef存储的就是以上东西
### 全局程序集缓存
1. GAC：多个程序访问的东西放在的共有的目录
### 强命名程序集能防篡改
1. 验签过程（**安装到GAC时**,GAC之外每次都验签）：
   1. 对PE文件进行哈希
   2. 公钥解密
   3. 对比结果（因为没有私钥就不能一毛一样地加密，所以安全）
### 延迟签名
1. 过程：
   1. 留存加密哈希
   2. 避免GACcheck
   3. 获取私钥并恢复验证
### 私有部署强命名程序集
1. 原因：
   1. 依赖
   2. 占空间
2. 方式：
   1. 私有部署
   2. 局部共享
### “运行时”如何解析类型引用
1. 如何找到类型(System.Console.WriteLine)：
   1. 准备着手解析Main
   2. 在IL中得到MemberRef位置
   3. MemberRef->TypeRef
   4. TypeRef->AssemblyRef（**如果是别的Assembly**）
   5. 找到程序集，现在该由上向下找了!
      1. TypeRef指向
         1. ModuleDef：直接创建
         2. ModuleRef：检查ModuleRef并加载文件
      2. TypeRef指向的是AssemblyRef
         1. 加载程序集清单文件，ExportedTypeDef指向
            1. 清单：直接创建
            2. 别的Moudule：加载文件并创建
2. 注意！
   1. Def表的指向是由上向下的（Type->Member）Module/Type/Method/Field/Param/Property/Event...真的多。
   2. Ref表的指向时由上向下的（Member->Type->Assembly）Member/Type/Module/Assembly
### 高级管理控制（没看）

## 类型基础
### 所有类型都从System.Object派生
1. `Object`包含的方法：
   1. `public`
      1. `Equals`
      2. `GetHashCode`
      3. `ToString`
      4. **`GetType`**
   2. `protected`
      1. `MemberwiseClone`
      2. **`Finalize`**
2. **`New`操作符做的事**
   1. 从托管堆分配内存（整个类型+Overload的容量）
   2. 初始化**类型对象指针**和**同步块索引**
   3. 调用构造器
### 类型转换
1. `GetType`总是知道对象是什么？
   1. 可以`new GetType()`，但是在`is as`的时候还是标准的`GetType`
2. 基类型可以灵活转换，派生类型只能显式转换
3. `is`和`as`
### 命名空间和程序集
1. `using`的作用
   1. 少打字
   2. 创建别名，用于处理歧义问题或者只导出命名空间中一部分简单命名
### 运行时的相互关系
1. 栈从高地址向低地址
2. 类型对象包含什么？
   1. **索引块**
   2. **类型对象指针**
   3. 静态成员
   4. **方法表**
3. 实例对象包含什么？
   1. Overload
   2. 实例字段
4. 非虚调用会导致从实际对象类型向上回溯
   1. 那么虚调用呢？
5. 类型对象的指针指向的是？
   1. **`System.Type`**，他们的关系**不是继承关系，而是类和实例的关系**

## 基元类型、引用类型和值类型
### 编程语言的基元类型
1. 基元类型的例外：
   1. new和non-new的冲突：
      1. 基元类型不必非要`new`
   2. 没有用`using System.Int32`的冲突：
      1. 编译器假定他们用了
   3. 类型转换例外：
      1. 编译器在不存在派生关系的时候设定了特殊规则：满足安全（不会发生数据丢失即可）
   4. 字面值支持
      1. 被看成是类型本身的实例
2. `check`和`unchecked`基元操作
   1. 操作符、语句和`编译开关`
### 引用类型和值类型
1. 造成引用类型效率下降的原因：
   1. 在托管堆分配
   <!-- 2. 需要初始化为0 -->
   3. 有同步块和类型指针
   4. 可能引发GC
2. 值类型的几个特征：
   1. 在栈分配
   2. 隐式密封
   3. 从`ValeType`派生
   <!-- 4. 字段也初始化为0.... -->
3. *推荐*值类型应用时的几个特征：
   1. 类型immutable（注意仅仅是推荐）
   2. 不需要继承、派生
   3. 尺寸较小
4. 值类型的几个特征：
   1. 值类型中不能*定义*新的虚方法
   2. 值类型赋值是复制内容
   3. 值类型变量之间无法直接影响
   4. 值类型直接被释放*无需垃圾回收*
5. 装箱的步骤：
   1. 托管堆分配 内容+overload 的内存
   2. 复制
   3. 返回对象地址
6. 拆箱的步骤
   1. 获取已装箱之中的各字段地址
7. 拆箱+转型注意：
   1. 对对象进行拆箱只能转换为原始值类型，不允许直接转型为兼容类型
8. 注意值类型调用方法成员的问题：
   1. 值类型不能定义新的虚方法，但是确实存在虚方法比如GetHashCode
      1. 如果重写了这类方法，那么不会装箱（想必方法的第一个隐藏参数就是这个值类型）
      2. 如果没有重写这类方法，即调用了基类的方法，那么会发生装箱（因为基类方法要的是object类型的this参数）
   2. 所以基于以上，*转换为接口需要装箱*，因为*接口变量必须包含对堆对象的引用*
9. 值类型调用定义好的接口方法不需要装箱，但是一定要注意参数是否装箱...应该是IL做了处理把object this变成了struct this
10. 方法调用时选择什么方法签名取决于*变量类型*，理解接口类型的方法定义的时候一定要把接口和类型视作两个东西。
11. 接口更改值类型字段（没看）
12. 对象相等性顺序：
    1.  参数是否null
    2.  引用是否相同（这个是提升到前面提升性能的）
    3.  类型相同
    4.  该类型所定义的字段相同（提升）
    5.  基类型所定义的字段相同
13. 静态方法ReferenceEquals
14. 值类型*需要给所有的字段进行初始化*，否则会报错“使用了未赋值的局部变量”。当然如果你用new相当于你知道都初始化为0这事，也就是初始化了
### `dynamic`基元类型
1. dynamic的runtime灵活特性可以体现在它一个变量可以承接很多不同类型的数据（比如在for中可以变化的）。为了实现它，其实它就是object
2. 反射的使用方式是：`GetType()->GetMethod()`。然后调用的时候`Invoke()`


## 出错的问题
1. `null as SomeType` 的时候会不会抛出异常
   1. 不会，`as`永远不会抛出异常
2. 基元类型是什么？（忘了由什么引入的了），Int32等基元类型的加法是不是借助于运算符重载？
   1. 编译器直接支持的类型，直接映射到FCL中存在的类型。编译器非常熟悉，以至于例如`Int32`的+运算csc不需要类似`Vector3`的方法调用`System.Numerics.Vector3::op_Addition`，而是直接使用ilcode `add`。
   2. 这里有一个逻辑上的不一致，但是应该也是一种折中。毕竟常用的基元类型也进行function call效率是有点低了。
3. 拆箱到底会发生哪些异常（隐含）
   1. Attempting to unbox *null causes* a `NullReferenceException`. 
   2. Attempting to unbox a reference *to an incompatible value type* causes an `InvalidCastException`.
4. 看以下代码
   ```Csharp
   struct Point{int i;}
   Point p1, p2;
   //赋值过程省略...
   p1.Equals(p2);
   ```
   这里是否发生装箱？
   1. `p1.Equals(p2)`会发生两次装箱：
      1. `Equals()`在`System.ValueType`中定义，`ValueType`为引用类型，由它实现的`Equals`在这里要求一个引用类型的`this`指针传递堆对象的引用，所以发生装箱。
      2. `Equals(object)`的参数类型是`System.Object`，所以`p2`也需要装箱
5. 类型的成员都有什么，类型算不算类型成员，委托算不算成员
   1. 类型成员：
      1. 名词：
         1. 常量（*静态*）
         2. 字段
         3. *类型*
      2. 动词：
         1. 构造器（静态/实例）
         2. 方法
         3. 属性
         4. 运算符重载
         5. *转换操作符*
         6. 事件
   2. 类型属于类型成员 “类型中可定义0个或多个以下种类的成员...类型可定义其他嵌套类型”，“定义类型的成员（包括嵌套类型）时，可指定成员的可访问性”
   3. 委托属于类型成员，委托实际上是类型
   4. 话说回来，一个纯粹连类型都可以是类型成员的语言，那想必一般实体都应该是类型成员。

## 自己的问题
1. 运算符在CLR中是怎么支持的？
2. dynamic的底层实现原理到底是什么？
3. 值类型和引用类型的实例方法的this有没有差别？

## 类型和成员基础
### 类型的各种成员
1. 类型的成员都有什么？
   1. 名词：
      1. 常量（*静态*）
      2. 字段
   2. 动词：
      1. 构造器（静态/实例）
      2. 方法
      3. 属性
      4. 运算符重载
      5. *转换操作符*
   3. 混合：
      1. *类型*
      2. 事件
2. 非嵌套类型可以设定internal和public，嵌套类型恐怕有更多
### 类型的可见性
1. 友元程序集
   1. 是什么？
      1. 定向的，可以访问本程序集`internal`*类型*的程序集
   2. 如何设置友元程序集？
      1. `System.Runtime.CopilerServices.InternalsVisibleTo(名称，公钥)`
   3. 编译时需要设定/out或者/moduleassemblyname开关
### 成员的可访问性
1. 可访问性：
   1. private：类型或*嵌套类*
2. 接口成员都是public
3. C#不能继承修改访问性，CLR可以放宽（为什么？）
### 静态类
1. static只能用于类不能用于结构（因为结构总是可以实例化）
2. 一些限制：
   1. 只能从`System.Object`派生（为什么静态类不适用继承）
   2. 不能实现接口（接口方法需要实例）
   3. 只能有静态成员
3. 会被直接标记为abstract、sealed抽象密封
### 分布类、结构和接口
1. `partial`关键字
2. 意义：
   1. 版本控制——*无需merge*即可同时修改一个类型
   2. *同一个文件*中拆分逻辑单元
   3. *不同文件*中拆分逻辑单元
3. 组件软件编程CSP
   1. 它似乎指的是不同公司合作进行OOP的一种实践，所以需要更独立的版本控制、文档、依赖处理、标识、安全性，和稳定的版本号以及对象模型等等。
      1. 主、次、内部（build）、修订（revision），后两个号的变化暗示着向后兼容
4. 组件*版本控制*关键字（全部可用于*方法/属性/事件*）
   1. `abstract`（不能构造实例/*如果构造必须重写*成员/x）
   2. `sealed`（不能被继承/不能被重写，只能用于虚方法/x）
   3. `new`（没有任何关系（用于嵌套类型）/x/x）
   4. `virtual`（x/可被重写/x）
   5. `override`（x/正在重写/x）
   注意区分abstract和sealed
5. `call`和`callvirt`
   1. `call`
      1. *可以调用静态方法*，对虚方法以非虚方式调用
      2. 并不进行null检查（这也是为什么可以调用静态方法的原因）
      3. 调用时上溯方法（静态方法也会被继承）
   2. `callvirt`
      1. 不可以调用静态方法
      2. check null，抛出异常
      3. check对象实际类型
   3. 不是虚方法C#也一定会用`callvirt`，因为check null提供了额外的安全（？）
   4. 但尼玛有时又需要用call，比如子类虚方法调用父类同名，否则会出现递归调用
   5. 值类型C#又倾向于使用`call`，因为*密封*不必多态并且*一定不为null*。并且虚方法调用方式*需要类型对象引用*，需要装箱。（哦我的上帝）

## 常量和字段
### 常量
1. 注意区分常量和只读
   1. 常量：编译器将在确定常量后将值保存到程序集*元数据*中。
      1. 似乎是以字面值而非构造器中的赋值（那是`readonly`）
      2. 但是它也是field： `.field public static literal int32 a = int32(0x00000005)`
      3. 问题就是dll中的`const`并不会影响调用它的程序集，所以如果希望有依赖，这里推荐使用`readonly`
   2. 只读：
      1. 只有*构造器*才可以写入
2. 所以只能定义编译器识别的*基元类型*的常量
3. C#支持非基元类型的常变量，只要设为`null`
### 字段
4. 加载类型对象是在方法首次进行JIT编译的时候，可以视作是要进入函数之前先安排数据
5. 字段修饰符：
   1. `static`
   2. `instance`（默认）
   3. `readonly`：除构造器外任何*方法*不可以写入
   4. *`volatile`*：读写操作不能从寄存器处理，顺序不可颠覆
      1. 写是最后一个
      2. 读是第一个
      3. 只有一些基本类型才可以做
6. 内联初始化（直接在变量定义处赋值，而非构造器处）只是对构造器的一种语法的简化。

## 方法
### 实例构造器和类
1. 没有被构造器构造的都保证是`null`或`0`
2. 构造器永远不能被继承，因为没有毛的意义
3. 内联初始化的构造器代码会在*每一个构造器中加入*->*调用基类构造器*->调用自己构造器的代码
4. 构造器*委托调用*可以减少生成的代码
### 实例构造器和结构
1. 性能考虑->CLR不会为*引用类型的每个值类型*都主动调用构造器，但
   1. 在引用类型中的嵌入的值类型保证为`0`或`null`。
   2. 在栈上的保证置零（一般没法保证）或要求读取前赋值
      1. 值类型不允许定义无参构造器，目的就是让开发者死了值类型会被自动调用无参构造的心
      2. 所以它也*不允许内联初始化*实例字段
      3. 但是*静态字段*可以内联初始化，因为它只用调用一次，没有性能压力
2. 值类型并不需要有构造器，但可以定义
3. 值类型*构造器*必须对所有成员完整赋值（即使是用this = new SameType()进行初始化也可以）
### 类型构造器
1. 默认没有，定义*只能定义一个and无参*
2. 默认私有，不能加修饰符
3. 调用时机：每次编译方法时均会检查所有*引用的类型*，是否已执行构造器。如没有，则在*native code*中插入调用，否则跳过，和加载类型对象的时机当然一致
   1. 为了保证调用一次，还要*有一个锁*
4. 为什么说类型没有静态字段是从基类型继承的...因为它确实没必要再所谓“继承”实际上是拷贝一份
### 操作符重载方法
1. CLR不知道操作符，所有的无论是基元类型还是非基元类型的操作符都是CSC干的事情
2. 要求
   1. `public` 
   2. `static`注意
   3. 至少有一个参数和当前类型相同
   4. 会自动生成`specialname`标志（关键，它是以这个方法应用于运算符的必须条件）
3. 这里有个问题，++怎么算？哦，显式this（一个参数）
4. 核心数值类型为什么没有定义任何操作符重载方法？
   1. 因为调用代价，所以编译器会在代码中专门查找针对这些基元类型执行的操作，从而生成*直接操作这些类型的IL指令*
      1. 我的理解是它的这些运算符相当于*内联*了
### 转换操作符方法
1. 要求
   2. `public`
   1. `static`
   3. 参数或返回值至少一个相同
2. 隐式/显式转型修饰符，在`public static`之后
   1. `implict`：任何时候发现均进行隐式调用
   2. `explicit`：只有显式转换时才调用
3. *形式*：
   ```Csharp
   public staic implicit/explicit operator 目标类型(当前类型)
   ```
   元数据方法名：`op_Implicit()`或者`op_Explicit()`
### 扩展方法
1. 实际是静态方法，但允许以实例方法的形式调用
2. 要求：
   1. `public`
   2. `static`
   3. 第一个参数前+`this`
3. 不支持扩展属性、事件、操作符
4. 只能在*静态类*中定义
7. 需要有*文件级作用域*，亦即不能在嵌套类中
5. 冲突处理（如果有两个类型都定义了同样签名的扩展方法）：用静态语法调用
6. 不检查非空
8. 扩展方法应用了一个*ExtensionAttribute*，以及包含它的静态类，以及程序集
### 分部方法
1. 可以解决以类作为最小粒度，基类只能使用virtual function来释放给派生类修改行为的局限：
   1. 只能是非密封
   2. 不能是值类型
   3. 不能是静态方法
   4. 效率问题
2. 形式：
   ```csharp
   partial class 类型名
   {
      partial void 函数名(); // 相当于有函数声明就行了
   }
   ```
   另一个文件：
   ```csharp
   partial class 类型名
   {
      partial void 函数名() // 相当于必须要有函数实现
      {
         函数体
      }
   }
   ```
4. 分部方法如果没有实现，就完全不生成IL代码
5. 要求：
   1. *类*也要分部
   2. 返回`void`，也*不能有`out`参数*——不能有任何这个函数会返回什么从而影响整个调用流的可能
   3. 签名一致，*特性合并*
   4. 视为`private`，为什么？恐怕因为分布类可能定义可能未定义，对于使用者来说能否使用是不清楚的
   
## 参数
### 可选参数和命名参数
1. 命名参数的调用传递方式和struct initializer语法不同
   1. 命名参数是`:`
   2. initializer是`=`
2. 传参求值的方向是*从左到右*（直觉）
3. 规则
   1. 有默认值参数必须放在最后
   2. 默认值必须是*编译期常量*
      1. 所有基元类型
      2. 引用类型的`null`
      3. 值类型的默认0（`default`或`new`）
   3. 不要重命名参数变量
   4. 模块外调用默认值会出现版本控制问题（因为这些编译器常量会嵌入在Caller中，它的问题实际体现了*调用者不知道也不会报错*的情况，只不过结果*不同于预期*）
      1. 解决之道：默认值设置为`null`，在函数内部逻辑判断`null`并赋予默认值
   5. `ref`/`out`并无默认值
   6. 命名实参必须在尾部
4. 可选和默认值实际上是标记了特性`OptionalAttribute`和`DefaultParameterValueAttribute`
### 隐式类型的局部变量
1. `var`是编译器支持的特性
2. 规则
   1. `var`不能为`null`（那怎么推断类型呢？）
   2. 不能为参数和字段
   而`dynamic`上述均可，`var`只能局部变量
### 以传引用方式向方法传递参数
1. `out`和`ref`是一码事，只不过csc会以不同标准验证你的代码
2. 传引用的实参类型必须形参类型一毛一样（为了保证类型安全，否则方法里可以把传进参数转为任意对象）
### 向方法传递可变数量的参数
1. `param`
2. 原理就是构建了一个数组
3. 规则：
   1. 最后一个参数可用
4. 注意调用成本：毕竟数组需要在*堆上分配*并GC
### 参数和返回类型的设计规范
1. 一般将参数类型设定为最弱的类型，通用（跟下面原因一样）
2. 返回值类型设定为最强的类型，通用（因为强->弱不差钱）

## 属性
### 无参属性
1. 元数据中会生成属性定义项...也是，否则intellisense怎么get到这是个property还是field/method呢？
2. AIP自动实现的属性：
   1. 就是`get`和`set`体不实现就好
   2. 但这玩意除了省了点打字的事之外感觉挺傻逼的
3. 对象和集合初始化器（**Object and Collection Initializer**）可以用于ref object：
   1. 形式
      ```csharp
      String s = new Employee(){Name="Jeff", Age=45}.ToString().ToUpper();
      ```
      这里仍需要用到`new`和类型，只不过可以把多个语句的内容放在一起罢了
   2. 除了用于实例化的时候设置*公共属性（或字段）*（调用的是`replace`），还可以用于`IEnumerable`和`IEnumerable<T>`接口类型对象的初始化（调用的是`add`）
      1. 需要多个实参的`add`方法可以通过大括号嵌套来调用initializer
4. 匿名类型（**Implicitly Typed Local Variables**）
   1. 形式：
      ```csharp
      var o = new{Name="Jeff", Year=1964}
      ```
   2. 构建的实际类型：
      1. 自动合成名称、构造函数、`Equals()`、`GetHashCode()`、`ToString()`
      2. *readonly field*
         1. property也确实只有`get`
   3. 进阶形式
      ```csharp
      String Name = "Grant";
      DateTime dt = DateTime.Now;

      var o2 = new {Name, dt.Year}
      ```
      这里o2会：
      1. 利用变量的变量名来设置名称
      2. 利用变量的值来设置值
   4. 源代码中的多个结构相同（类型+名称+顺序）的匿名类型会生成为一个。
5. *但是由于不知道名称，所以没办法作为参数和返回值（dynamic了？）*
6. Tuple的最复杂形式也只有8个item，所以最后一个item可以给另一个tuple
### 有参属性
1. 形式：
   ```csharp
   public Boolean this[Int32 bitPos] {get{} set{}}
   ```
   注意，*get的实参在方括号内设定了，set的实参就是value*
2. 索引器*可以重载*，只要签名不同
3. `get`和`set`的访问修饰可以不同，背板修饰是限制较小的，特定修饰是较大的
4. 属性不能带有“与包容类型的泛型类型参数不同的自己的泛型类型参数”，原因是“属性”没有行为，不应该随类型改变而改变“行为”

## 事件
1. 步骤：
   1. 定义事件*附加信息*
      1. 继承`EventArgs`
      2. 类名以`EventArgs`结束
   2. 定义*事件成员*
      1. 形式：
         ```csharp
         public event EventHandler<NewMailEventArgs> NewMail; // 注意这里的EventHandler<参数>类型，还有前面的event
         ```
      2. 由于`EventHandler`的形式是：
         ```csharp
         public delegate void EventHandler<TEventArgs>(Object sender, TEventArgs e) // 显然delegate也是类型成员
         ```
         意味着reciver必须定义如下形式的回调：
         ```csharp
         void MethodName(Object sender, NewMailEventArgs e);
         ```
      3. 事件的修饰符是`public`，但是在Unity中的`UnityEvent`则不用
   3. 定义引发事件的方法：
      1. 形式：
         ```csharp
         protected virtual void OnNewMail(NewMailEventArgs e){
            EventHandler<NewMailEventArgs> temp = Volatile.Read(ref NewMail); // 出于线程安全考虑，先复制
            if(temp != null) temp(this, e);
         }
         ```
   4. 定义方法将输入转化为期望事件
      1. 说白了就是调用`OnMail`的方法
### 编译器如何实现事件
1. 底层机制：
   1. 两个数据结构：
      1. 私有委托
      2. 公有`add`和`remove`委托的方法
         1. 其中使用了经典的`interlocked.exchange`范式，即
            1. 构建`newValue`
            2. 拿到`oldValue`
            3. 开始循环，循环中调用`interlocked.exchange`，调用完后在循环判断中判断是否修改成功，如果没有继续循环（*循环中记得也要构建`newValue`*，毕竟别人修改之后你得把别人的修改也带上）
      3. 注意`BeginInvoke()`和`EndInvoke()`是在delegate中实现的
2. 此处编译器也会在元数据中生成记录项，仅仅是为了建立“事件”和这些method及field的联系
### 设计侦听事件的类型
1. C#编译器将+-事件翻译成`add()`和`remove()`
2. 一旦登记事件便无法被垃圾回收（是因为有一个引用吗？）所以`Dispose()`方法中应该取消对事件的关注
### 显式实现事件
1. 用于处理有很多事件的情形，避免很多事件的委托字段占用大量内存，并且在继承、实例化的时候都要占据这么多内存
2. 核心数据类型：
   1. 使用*字典*来中心式管理事件`Dictionary<EventKey, Delegate>`
   2. 实现`Add()`和`Remove()`
      1. 两者都需要互斥
      2. `Remove`需要对`Eventkey`是否存在来进行判断，如果不存在，直接删除字典条目
   3. 实现需要`EventKey`的`Raise`
      1. 这里需要`Delegate.DynamicInvoke`以便在运行时查证参数的正确性
      2. 至于使用它的类，核心就是一个`public event EventHandler<FooEventArgs>`的property（和非显式实现的形式一致）。这个property**和自动实现的event的差异就是具有显式的`add`和`remove`**，和AIP的差异就是不是`get`和`set`。实现中调用前者的`Add`和`Remove`，对`FooEvent`事件的条目进行添加
      3. 然而，那如何去区分不同的`EventKey`呢？实际上在类中需要定义两块*相互对应*东西，一块就是继承并新建一个`XXXEventKey`类型，然后在前面所述的`EventHandler<FooEventArgs>`的property之中调用相关的`Add()`时把EventKey传入
## 泛型
1. 泛型的优势：
   1. 源代码保护——和C++模板相比
   2. 编译器类型安全？
   3. 更清晰的代码——减少强制类型转换
   4. 性能更好——没有装箱和垃圾回收
      1. 于是乎对于引用其实性能提升没有那么明显
### 泛型基础结构
1. 泛型也是类型——有类型对象
2. 开放类型：类型参数没填完，不能实例化
3. 每个封闭类型都有*自己的*静态字段和静态方法，不会共享
4. 指定类型实参不影响继承层次结构
   1. 即实参改变，泛型类型的爸爸不会变
5. **设计一个可以装任何东西的链表**：
   1. 设计一个结构基类，包含next什么的
   2. 设计一个泛型*派生类*，包含实际的data成员
   毕竟即使节点类型不同，但都是引用类型，就指针么。这样说起来其实C++也完全可以做hhh
   1. 这里其实暗含了一个假设，就是每个`Node`都是一个引用，所以非泛型基类可以用同种方式进行next
   2. 那么如果使用值类型呢？
      1. 值类型无法继承...
6. `using`指令除了给其他命名空间的类型指定别名之外的别的别名用途：
   ```csharp
   using DateTimeList = System.Collections.Generic.List<System.DateTime>
   ```
   这样在简化语法的同时还避免了*另外定义一个类*对同一性和相等性的破坏
   1. 同一性和相等性的破坏指的是如果直接继承特化的泛型类型，并将之确切化但不赋予任何实现，两个类型则不相等。
7. 代码爆炸：
   1. 泛型类型生成的实际操作代码当然是Native Code，而代码爆炸也存在于其中中，缓解手段：
      1. 同一个AppDomain中的*相同类型参数组合只编译一次*。
      2. 所有引用类型的类型参数都是一种东西
### 泛型接口
1. 在继承接口的时候接口的类型参数可以保持未指定也可以指定
2. *我可以继承没有指定类型参数的泛型类型嘛？应该是可以的*
### 泛型委托
1. 委托是类型——泛型委托正是自动合成的泛型类型
### 委托和接口的逆变和协变泛型类型实参
1. 实际上逆变和协变都基于一个事实：派生类可以填充基类，如果你需要一个基类但是给了你一个派生类这并无不妥。
   1. 所以，输入，允许是派生类
   2. 输出，允许输出出来基类
   3. 所以对于委托，一个满足1、2的委托b可以被一个原始函数a给赋值。因为对*真正使用b委托的用户（也就是a委托给了b）*来说，完全是无缝的，不会存在填充不满的东西。
2. 至于委托和接口的类型参数声明协变和逆变（在类型参数前）：
   1. 逆变量（泛型参数可以从一个类更改为它的*某个派生类*）就是可以转换为派生类（意思是我有一个输入为基类的委托，我可以把它赋值给一个派生类输入委托，因为前者的基类本身调用就没问题，后者派生类输入也可以填充基类的数据结构），*标记为*`in`
   2. 协变量（泛型参数可以从一个类更改为它的基类）就是向基类去了，*标记为*`out`
3. 它和前者实际上是一样的，简单来记忆就是被赋值者是使用者
4. **只有编译器能验证类型之间存在引用转换，这些可变性才有用**，意味可变性这一步是在编译阶段。
   1. 值类型由于需要装箱，所以不存在引用转换，所以没办法利用这一点
      1. **为什么？**
5. 按道理说应该编译器自动实现这个所谓逆变和协变的除了增加复杂度外没有屁用的东西啊？
   1. C#的哲学是，我可以支持，但你要*订立协定*（貌似是，本来C#就已经很复杂了，这些隐式的自动机制最好还是越少越好，才能给用户保证一定的可配置性）
### 泛型方法
1. 调用方法时进行**类型推断**，但是注意，类型推断也是在*编译时进行的*，通过*变量类型*来推断
   1. 也是，除了引用类型，*我肯定是要在native code之前知道你的泛型类型的，否则我怎么生成对应的/不同的操作指令*？但是JIT的时候来判断不行？
   2. 既然如此，我那个时候能知道什么？我在不执行Native code的时候我能知道什么？只有字面类型，除非我执行一下native code来判断这个变量的类型...但是我这个native code哪来的？我怎么写判断值类型的native code？
2. 实际上这里存在一个逻辑就是，模板的类型推断方式，应该都是在编译期进行的
### 泛型和其他成员
1. *属性、索引器、事件、操作符方法、构造器和终结器*都不能有自己的类型参数
### 可验证性和约束
1. C#编译器也会对代码的泛型调用方法进行**验证**，验证代码适用于当前所有类型和未来任何类型。（[之前](#nativeveri)是IL->Native的验证）
   1. 如果并非所有类型都能进行定义中的方法调用，而又没有合理的约束，C#编译器会报错
2. 形式：
   ```csharp
   where T : Icomparable<T>
   ```
3. 继承泛型虚方法*不能重写约束*
4. 约束类型：
   1. **主要约束**：类型和派生类型、`class`和`struct`（可空值类型不满足struct）
   2. **次要约束**：**接口**类型
   3. **构造器约束**：类型参数是实现了*公共无参构造器的、非抽象类型*
      ```csharp
      where new()
      ```
5. **所有值类型都隐式提供了公共无参构造器**（但是调用不调用是另一回事）
6. 验证问题：
   1. 泛型变量第一次转型除object外非法，除非约束兼容（因为无法保证满足强制类型转换条件）
   2. 除非约束为引用，否则赋值为`null`非法；但和`null`比较合法
   3. 不同类型的变量相互比较非法
   4. 不能将类型参数约束为*具体值类型*，因为没屁用
   5.  很遗憾，*运算符无法应用于任何泛型变量*
      1.  虽然基元类型可以支持这些运算符，但是原因是因为编译器认识
      2.  无法支持这些运算符的原因是没有这种“给我保证实现了这些运算符”的约束...或许可以有一个这样的接口约束？
7. `default`关键字：引用设为`null`，值类型设为`0`

## 接口
### 类和接口继承
1. 类继承，不仅继承签名，还继承所有实现
   1. 多继承的“缩水版”实际上指的就是接口继承*不继承方法实现*，必须自己实现
### 定义接口
1. 成员：
   1. 可以定义*事件、属性*（反正你也要自己实现）
   2. 不能定义字段、构造器、静态成员
2. **接口用`interface`关键字声明**
3. CLR把接口视作类型（但是显然有区别，因为可以继承多个接口）
4. 接口与接口之间的“继承”也是协定关系
### 继承接口
1. 实现接口的方法时的修饰：
   1. 要求：
      1. `public`
   2. 默认
      1. `virtual sealed`
2. 使用接口变量来调用实现者，**会调用其当下类型中的那个实现，而非基类实现**
   1. 原因应该是每个类型对象都会有一套接口的基指针，如果是父类实现了的接口应该就不一定有，恐怕要上溯基指针，call的时候也有可能是....
   2. 父类的接口`virtual`方法应该是直接继承下来的。最后的callvirt确实是call的父类
### 关于调用接口方法的更多探讨
1. 接口类型也是继承了`Object`，所以接口可以调用`ToString()`等
2. *变量的类型规定了能对这个对象执行的操作*
3. 值类型对象应该也有方法表
4. 操，**类型的方法表中包含**：
   1. 继承的所有类/接口的*虚方法*——它必须要在那里，哪怕没有重写都要在那里，那我才能实现同一个entry的多态机制
   2. 引入的*新方法*——毫无疑问
   意思是说，找父类的方法，真的就只能上溯
5. 接口名称作为方法名的前缀就是显式定义接口方法EIMI，
   1. 此时并不允许`public`，只有默认的`private`
      1. 表明只有*接口变量*才能够调用这个方法（这和private的语义有点冲突啊？有冲突）
6. EIMI方法*不能被重写*
   1. 这表明了这一类方法确实*不是对象模型的一部分（不能继承）*，应该只是一种纯粹的“组合”，视作新实现的非`virtual`的`private`方法。
7. 泛型接口的优势：
   1. *编译期类型安全*
   2. *效率高*
8. 如果某个值类型实现了非泛型接口，如IComparable，那么：
   ```csharp
      SomeValue a,b;
      a.Compare(b);
   ```
   在这里，a调用Compare的时候不发生装箱。但是传入b的时候会发生装箱。
9.  *C#编译器为接口约束生成特殊的IL指令导致直接在值类型上调用接口方法不用装箱*...实际上是带有接口协定的泛型类型参数使得可以直接在该类型参数的形参上直接调用接口方法（而不用先转为Object再到接口类型才调用），从而减少了装箱
### 显式接口方法实现来增强编译时类型安全性
1. 实际上就是用类型特化方法来在变量类型是当前值类型的时候替代同名*非泛型*接口方法。同时给同名接口方法一个非类型特化实现
2. 用于处理按`该类型变量.接口方法(实参)`的形式调用接口方法时避免非泛型接口（因为有的时候只有非泛型接口）导致实参装箱的问题
### 谨慎使用EIMI
1. 问题
   1. intellisense不能用
   2. 需要转换成接口才能调用，而转换成接口的时侯会装箱（因为3不能由原始变量直接调用，所以需要转型成接口）
   3. 不能由派生类调用...甚至不能由原始类型变量来进行调用
### 基类还是接口
1. 动词和名词
2. 基类重用的多就基类，接口更为灵活就接口

## 字符、字符串和文本处理
### 字符
1. Char包含：
   1. 两个公共只读常量
   2. 存储为一个16位的Unicode代码
2. Char静态：
   1. Category
   2. Is
### System.String类型
1. 神奇了，`String`这种基元类型并不是在栈上，*而是在堆上，是引用类型*，那自然会进行GC。
   1. 但是它基元的体现是，编译器允许在源代码中直接使用字面值字符串
   2. 但是另一个神奇的点在于，*字符串又是immutable的*！
      1. 优势：
         1. 操作不改变原字符串...
         2. 线程同步无需考虑，从而允许字符串合并——留用
2. `@`用于声明非转义的逐字字符串
3. 密封类（基元类型为了避免破坏都有可能密封）
4. 比较字符串就各种语言文化敏感啦啥的
5. *字符串留用*：
   1. 哈希表
      1. key：字符串
      2. value：托管堆`string`对象的引用
   2. 方法：
      1. Intern
      2. IsInterned
   3. 字面值字符串默认留用——现在禁用了MB
   4. 用法：
      1. 假设要比较的所有字符串已经留用
      2. 留用将要比较的单词
      3. 于是可以用`Object.ReferenceQquals`来比较
   5. 感受：你完全可以实现你自己的字符串留用机制
6. *字符串池*
   1. *元数据中只出现一次字面值，其他的都引用它*
      1. 啥玩意？如何实现的？
### 高效率构造字符串
1. StringBuilder
   1. 原理：内部维持了*Char数组但可变*
   2. 功能：增删改查
### 获取对象的字符串表示：ToString()
1. `Object.ToString()`只是返回对象所属类型的全名
2. 如果输出需要指定语言文化，使用`Object.ToString()`就是不完善的，因为它默认使用与调用线程关联的文化信息
   1. 要实现`IFormatable`才行
### 编码：字符和字节的相互转换
1. 使用`System.Text.Encoding`
   1. 这里有UTF8、UTF16和本地代码页方法
   2. 过程是编码成字节数组、或者从字节数组解码成字符串
2. 字节流的编解码：
   1. 前面所说的Encoding不记录状态，流传输读了半个字符就很傻
   2. `Encoding.GetDecoder`
   3. 机制：
      1. 尽可能多解码字符
      2. 剩下不够解的保存下来
      3. 下次再来的时候连接上去
### 安全字符串
1. `System.Security.SecureString`
2. 机制：
   1. 使用*非托管内存——避免垃圾回收*
   2. 在进行任何增删改查的时候*解密->处理->加密*
   3. `Dispose()`可以清除非托管内存的缓冲区

## 枚举类型和位标志
### 枚举类型
1. 特点：
   1. 枚举类型是*值类型*
   2. 强类型
   3. *基元类型*
      1. 可以用各种操作符来操作
   4. 不能定义任何方法
2. 结构：
   1. 一组`const int`
   2. 一个当前值
3. 运行时不需要符号，编译时才需要
4. 可以声明基础类型是`byte`, `sbyte`, `short`, `ushort`, `int`等等
   1. 语法
      ```csharp
      internal enum Color: byte{
         While, 
         Red,
         Black
      }
      ```
5. 枚举类型符号数值可以相同...
6. 可以返回所有Enum数值组成的数组，数组类型是`Color[]`，所以可以直接输出
### 位标志
1. 枚举类型表示单个数值，位标志表明位集合
2. 需要应用定制特性`FlagsAttribute`
   1. 当应用该特性时，`ToString()`会把数值拆分
      1. *降序排列数值*（*先处理All*，如果有了就不用搞了）
      2. 和实例进行按位与，如果结果ok，则视作存在，附加到字符串，并将该数值位置置0
      3. 如果最后还不为0，则直接输出原始数值
      4. 否则输出组合
3. `Enum`也可以`Parse`
4. `IsDefined()`时输入一串以逗号分隔的字符串时不行，因为它会视作一个key进行处理
### 向枚举类型添加方法
1. 扩展方法么

## 数组
1. 始终是*引用类型*
   1. Object->Array->XXX
2. 保存值类型的时候是未装箱内存块
3. 保存引用类型是引用
4. 开销信息：
   1. 维度
   2. 下限
   3. 长度
   4. 元素类型
### 初始化数组元素
1. *数组初始化器*
   1. 注意区分值类型初始化器，那个可能要指定变量名的，这个不需要
2. 初始化可隐藏各种类型，但是元素类型必须一致
### 数组转型
1. 引用类型只要有隐式/显式转型符即可转型（*维度一致*）
2. 值类型不能直接转换和为元素赋值
   1. 可以用Array.Copy
      1. 值->引用 装箱
      2. 反过来拆箱
      3. 加宽类型
### 所有数组都隐式派生自System.Array
1. Array提供了`Clone`、`CopyTo`、`Length`等等
### 所有数组都隐式实现IEnumerable、ICollection和IList
1. 一维数组会实现泛型版本
   1. 会在“CLR创建这个类型的时候”，为数组存储的这个类型及其基类实现这些接口
   2. 于是允许了该数组在任何需要IE、IC、IL的时候可用
   3. 但是如果数组包含的是值类型元素则只会实现该类型接口（因为值类型和引用类型内存布局不同）
2. IList是随机访问接口、IEnumerable是foreach接口、ICollection是增删接口
### 数组的传递和返回
1. 数组参数传递时传递的是引用
   1. 如果不想修改数组最好进行Copy
   2. 但是Copy也是浅拷贝
### 数组的内部工作原理
1. 0基数组循环的时候只进行一次下限和上限(也只检查一次！)检查
2. 非0基数组和多维数组都要将check从循环中拿出来
3. 不安全的访问方式将关闭上下限检查并且基于指针进行处理。
   1. 需要fixed掉那个数组的指针
### 不安全的数组访问和固定大小的数组
1. 非托管堆可以用`stackalloc`*语句*来分配内存块，使用不安全的指针来进行操控，不能将这个缓冲区的地址传给大部分FCL方法。
   ```csharp
   Char* pc = stackalloc Char[width] // 需要指定类型才能做地址偏移嘛
   ```
   1. 要求：
      1. 一维0基
      2. 仅包含值类型
      3. 值类型中还不能有任何引用类型的字段（怕是它不归GC、移动等管理...废话，指针的值是fixed）
2. `struct`之中也可以嵌入
   ```csharp
   internal unsafe struct CharArray{
      public fixed Char Characters[20];
   }
   ```
   1. 要求：
      1. 数组fixed，一维0基
      2. 仅包含基元类型中的一部分值类型

## 委托
### 初识委托
1. 委托可以调用私有方法，只要你“委托了”
### 用委托回调静态方法
1. 委托的逆变性和协变性跟前面一样，但只有引用类型才支持
### 用委托回调实例方法

### 委托揭秘
1. 注意委托也是要new的，毕竟也是一个类型的实例
2. 编译器自动将委托的声明定义为一个类型
   1. 继承关系`Object->Delegate->MulticastDelegate`
   2. 委托内部的实例方法：
      1. 构造器
         1. 参数——方法指针和实例引用
            1. 在写代码的时候并没有明确object，这个步骤是csc给你做的
      2. `Invoke()`
      3. `BeginInvoke()`和`EndInvoke()`
   3. 静态方法：`Combine()`和`Invoke()`
   4. Fields:
      1. `_target`
      2. `_methodPtr`
      3. `_invocationList`
### 用委托回调多个方法（委托链）
1. 建立委托链的中间过程会产生很多临时对象（可简单将委托视为immutable的），那些对象都会被GC
2. 调用委托链的时候会依序调用
3. 委托被调用只有最后一个结果可以保存
4. 可以通过`GetInvocationList()`来返回一个Invocation的列表从而自行进行调用控制
### 委托定义不要太多
1. Func和Action不能定义带ref和out参数的
2. *`params`、默认参数、泛型参数约束*等都需要定义自己的委托类型
### C#为委托提供的简化语法
1. 不需要非要`new`一个`delegate`的类型
2. 匿名函数就比较难受
### 委托与反射
1. 核心是`System.Delegate.MethodInfo`的静态`CreateDelegate()`方法
   1. 这个方法的最简单形式需要两个参数
      1. `Type delegateType`（就是你定义的delegate类型，就是你想创建一个什么类型的`delegate`，它显然需要和这个函数的签名是一致的，*估计是可以处理同名函数之中的重载*）
      2. `Object target`（参数0，也就是this指针）
   2. 调用时当然先要得到前面的`MethodInfo`，方法也很简单
      1. `typeof(某个type).GetTypeInfo().GetDeclaredMethod("函数名")`
   3. 然后就用这个返回的对象`CreateDelegate()`就好了。

## 定制特性
### 使用定制特性
1. 可以理解为定制你自己的`public`, `sealed`等等特性，这里编译器并不理解他的意义，只不过机械地生成了对应的元数据
2. 非位标志枚举类型可否按位与？非枚举除了输出样式外有没有什么别的骚操作？
3. 可以用以下形式表明特性应用的位置：
   ```csharp
   [assembly: SomeAttr("Kernel32", CharSet = Charset.Auto, SetLastError = true)]
   ```
   1. 注意后面的参数（增强型构造器）
      1. 不具名的参数传递是“定位参数”，是构造器的参数（必须给）
      2. 具名参数就像值类型的initializer一样是公共field或property（optional）
   2. 构造之后特性的*所有状态*都会被序列化到*目标元素*的*元数据表记录项*中
4. 定制特性也是一个类的实例，从`System.Attribute`中*直接或间接*派生（说白了是引用类型）
5. 特性当然也可以修饰特性类，比如用于修饰特性用法的`AttributeUsage`
### 特性构造器和字段/属性数据类型
1. 特性的*各类数据通常情况下都要在编译期能够求值*（字面值/常量表达式，因为需要序列化到元数据之中）。
### 检测定制特性
1. 特性仅被标记，没有相应的处理逻辑则没有屁用
2. 这个处理逻辑的基础是“反射”
   1. 在实际代码中，通过反射在运行时检查（基于*字符串比较*）特性的存在情况以及特征
      ```csharp
      this.GetType().IsDefined(typeof(FlagsAttribute), false)
      ```
   2. 然后基于检测结果，执行一些逻辑分支代码特性才能真正发挥作用
   3. 如果使用`GetAttribute()`的话，就会在调用处也构造获得的实例
### 两个特性实例的相互匹配
1. 进行`Equals`或者`Match`匹配的时候还不是要老老实实构造一个对象并且对比。但为什么不能直接对比字段呢？
   1. 哦，因为那个字段是private的
   2. 那为什么不能写property？当然可以，只不过书中认为这是“老老实实写代码”的方法。
### 检测定制特性时不创建从Attribute派生的对象
1. 使用`CustomAttributeData.GetCustomAttibutes()`，它会提供一些：
   1. `DeclaringType`——描述函数的参数、返回值
   2. `ConstructorArguments`——描述定位参数
   3. `NamedArguments`——命名参数
   4. 这些输出描述的方法
### 条件特性类
1. `ConditionalAttribute`可以用于修饰`Attribute`类型
   1. 只有在定义ConditionalAttribute之中指定的符号时才会在元数据中生成该特性信息。
   2. 但是该特性会被编译，只是不会进到使用者的元数据之中而已

## 可空值类型
1. 就是`strcut`
   1. 一个`hasValue`（默认是空）
   2. 一个`value`
### C#对可空值类型的支持
1. 声明和定义：可以用类型+`?`的形式来搞定
   ```csharp
   Int32? x = 5;
   Int32? y = null;
   ```
2. 允许转型、操作符
   1. 其他的只要有一个是`null`结果就是`null`
   2. 操作符`&`和`|`有差别：
      1. `&`选最拉胯的那一个
      2. `|`选最不拉胯的哪一个
      3. 拉胯指数排序：`false` > `null` > `true`（注意null只是中间拉胯的）
   3. 实际运算时的逻辑(`+`)：
      1. 检查两者是否为空
      2. 如果都不为空则返回结果
      3. 否则返回null
3. 自己定义的可空值类型也可以正常调用重载的操作符和方法
### C#的空接合操作符
1. 语法糖
### CLR对可空值类型的特殊支持
1. 注意是CLR，装箱时
   1. 如果是`null`直接装箱`null`
   2. 如果非`null`直接装箱`value`
2. 拆箱时可以直接给到`nullable<T>`
3. `GetType()`会将其视作`T`
4. 可空值类型可以直接转化为接口
5. 说白了就是把它当成value而非wraped value
   
## 异常和状态管理
### 定义“异常”
1. 当动词不能完成任务时，就应该抛出异常
### 异常处理机制
1. 过程：
   1. `catch`搜索会从栈顶到栈底进行搜索
   2. 当搜索到结果时*内层的`finally`块都会被执行*
   3. 执行`catch`成功后，它再执行同级别的`finally`块
   4. 然后执行`finally`块之后的语句
2. `finally`块是保证会执行的代码——并且在这里线程不会停止
3. 如果`catch`和`finally`块发生异常，那么CLR的异常处理机制仍然执行
   1. 只不过`try`中抛出的异常将会完全丢失
### System.Exception类
1. 包含：
   1. 一些自定义信息：
      1. `Message`
      2. `Data`（字典，可以在再次抛出的时候定义一些自定义消息）
      3. `HelpLink`
   2. 一些调用信息：
      1. `Source`（哪个程序集出错）
      2. `StackTrace`
      3. `TargetSite`（哪个方法出错）
   3. 一些异常信息：
      1. `InnerException`
      2. `HResult`（COM的异常处理方式，用于跨托管/非托管使用）
2. `StackTrace`属性
   1. 当`new Exception`的时候为`null`
   2. `throw`的时候CLR记录抛出位置
   3. `catch`的时候CLR记录捕捉位置
   4. 当`catch`内部块访问`StackTrace`属性的时候，*CLR的记录代码会被调用*，再行创建*异常抛出到捕捉位置*的所有方法
      1. 这里不包括任何比catch块更高的方法
         1. 如果需要，可以用`System.Diagnostics.StackTrace`
3. `throw`带有具名对象会导致CLR重置异常起点
   1. 而Windows无论是是否具名throw都会重置
4. CLR如果能找到调试符号，那么StackTrace之类的属性将包括*代码行号*（也是，*C#由于有元数据，PDB中就不用存函数名什么的了，内容会少一些。元数据之中没有的，需要存储的恐怕就是跟源代码有关的信息了*）
### 抛出异常
1. 这里有一个自动化处理异常序列化、反序列化的案例（没看）
### 用可靠性换取开发效率
1. 如果状态损坏很糟糕，那么就用
   1. `AppDomain.Unload`：`ThreadAbortException`会使得所有`finally`块工作, 404注释
   2. `Environment.FailFast`：只有继承CriticalFinalizerObject的对象的Finalize等会工作
2. 使用以下语句将会调用`finally`:
   1. `lock`：`finally`会释放锁
   2. `using`：`finally`会调用`Dispoe()`
   3. `foreach`：`finally`会调用IEnumertaor.Dispose
   4. 析构器：`finally`调用基类的`Finalize`
3. 线程中捕捉异常可以在另一个线程的EndXXX中抛出异常
4. 几种编程实践：
   1. 有的时候，一旦出现了任何异常，就代表事情没有被合理地完成。此时需要进行回滚并向上通知，这种情形下有两点：
      1. 捕捉所有的异常，进行回滚
      2. 重新抛出被捕捉异常（位置没有变化）
   2. 另外一种情形，就是作为调用者并不需要也并不应当知道内部实现逻辑，所以需要对异常重新包装：
      1. 捕捉一个对应异常
      2. 抛出一个更高层级的异常并将先前的异常设为`innerException`（类型和位置都发生了变化）
   3. 如果仅仅是在异常中添加内容：
      1. 捕获异常
      2. 在其`Data`属性中添加数据
### 未处理的异常
1. 应用程序应该建立处理*未处理异常*的策略，*类库开发者用不着去关心这个*
### 对异常进行tiaoshi
1. VS中可以勾选有些异常调试
   1. 指的是在被`catch`之前就被捕捉到
### 异常处理的性能问题
1. 如果使用返回值`true/false`或者`index`，将要面对的“两个世界的最坏的情况”是指：
   1. 你又要给`true/false`写逻辑
   2. 又要给catch写逻辑
2. `TryXXX`的形式是一种`true/false`规避频频调用的`catch`性能损失的方法，但这仅仅意味着这里的`false`处理的是一种失败，别的失败你还要在里面去处理

## 托管堆和垃圾回收
### 托管堆基础
1. 访问一个资源的所需步骤：
   1. 分配内存
   2. 初始化
   3. 使用资源
   4. 摧毁状态（实际是本地资源）以进行清理（一般不需要，就是那些`Dispose()`）
   5. 释放内存
2. NextObjPtr在new过程中的变化：
   1. 计算类型所需字节数
   2. 加上overload来分配
   3. 调用构造器初始化
   4. 检查是否有足够空间，分配对象，返回地址，`NextObjPtr += size`。
   这里暗示了托管堆的理想情况也是*线性扩展*
3. 发生垃圾回收的时间：
   1. new操作符发现第0代不够
      1. 如果只有0代超出，那只GC 0代
      2. 如果*因为0代刚分配完导致1代超出那么本次不会进行1代GC*，会顺延到下次
      3. 如果没有回收到足够的内存，就会执行完整回收
   2. 显式调用Collect（回收2代）
   3. *Windows报告（系统总体）低内存*（大收紧，回收2代）
   4. CLR正在*卸载AppDomain*（小关闭）
   5. *CLR正在关闭*（大关闭）
4. 根：所有*引用类型的变量*
5. *同步块索引*有1位在GC中的用处：
   1. GC扫描时该位设为`0`，表示对象应删除
6. 引用计数和引用跟踪：
   1. 引用计数会计数
   2. 跟踪只标明0/1
7. 跟踪如何处理循环依赖？
   1. 只会标记依赖了没依赖，不会标记几次
8. 垃圾回收的过程：
   1. *暂停所有线程*
   2. 标记所有堆对象*同步索引块*中的一位为0
   3. 检查活动根标记所引用对象为1
   4. 移动所有为1的对象紧靠
   5. NextObjPtr刷新
### 代：提升性能
1. 第0代垃圾很多就减少第0代预算
   1. 如果回收了一次发现垃圾很少就会增大第0代
   2. 如果没有回收到足够的内存，就会执行完整回收
2. `>85000`字节的对象是大对象（移动起来昂贵（1，2），一般寿命长（3））
   1. 分配在large object heap
   2. 不压缩
   3. 逻辑上视作第2代（只有触发2代回收的时候会回收）
3. 垃圾回收模式：
   1. 工作站
      1. 应用程序挂起短（估计是单核，不影响别的任务）
   2. 服务器
      1. 占用所有core进行*并发回收*（时间上并发）
      2. 每个core仅*负责一部分区域*（空间上分块）
4. GC两种子模式：
   1. 并发：
      1. 并发标记不可达对象
   2. 非并发：
#### 使用需要特殊清理的类型
1. 意义在于，有些资源如果不进行特殊清理会造成内存泄露
2. 存在Finalize对象会被提升至+1代，在GC后才专门用一个高优先级线程调用`Finalize`方法
3. `SafeHandle`继承自`CriticalFinalizerObject`
   1. 会立即编译`Finalize`
   2. 调用非Critical之后调用
   3. AppDomain被强行结束时也会调用
4. SafeHandle用于自动控制包装的本机资源
5. Dispose用于手动控制本机资源
<!-- 6. using可自动于finally中调用Dispose() -->
7. 有的时候存在*托管对象很小，托管的实际资源很大*：
   1. Add/RemoveMemoryPressure（添加实际内存压力）
   2. HandleCollector.Add（对该对象初始化最大计数，和添加计数）
#### Finalize的内部工作原理
1. 含Finalize的在*构造器被调用之前就会放在终结列表中*
2. 如果不可达，*回收之后会被放在freachable队列中*
   1. 并被*提升至更高级别*
3. *高优先级线*程专门调用这些对象的Finalize方法并移除之
4. 下一次清理高级垃圾的时候，所有的根都不引用这些托管垃圾了，他们就可以被安全清除。

5. 四个标志：
   6. `weak`
   7. `weakTrackResurrection`
   8. `Normal`
   9. `Pinned`
6. fixed会让CSC在对象局部变量上生成“已固定”标记，比采用代理进行非托管代码交换的形式效率要高一些

## CLR寄宿和AppDomain
1. 垫片与最新版本的的CLR相同。负责启动CLR服务器
   1. 然后CLR启动对应的AppDomain
   2. 类型对象不会由AppDomain共享
      1. 但是有些是AppDomain中立的类型，比如MsCorLib.dll，这些为一个进程所有AppDomain所共用
1. 按值封送和*按引用封送*要搞一搞
## 程序集加载和反射
1. 程序集在加载的时候CLR即是用`Assembly.Load`来加载程序集，传入的是程序的完整名称的字符串
2. LoadFrom(path)只负责从path中获取程序集全名，然后用全名调用Load，但Load会从GAC-根目录等位置开始查找，所以找到的并不一定是自己想加载的
3. 不执行构建但是加载的方式是`ReflectionOnlyLoad()`
4. 应用程序使用反射的情况之二
   1. 序列化——需要知道类型定义了什么内容
   2. 根据强命名加载程序集
5. 反射的问题：
   1. 依赖字符串，于是无法在编译期进行检查
   2. 依赖字符串对比，效率低
   所以更推荐使用类型及接口
6. Type和TypeInfo的区别
   1. `Type`是`TypeInfo`的轻量级引用

## 序列化与反序列化
1. 调用序列化器进行序列化反序列化的形式如下:
   ```csharp
   formatter.Serialize(stream, objectGraph);
   // ...
   object a = Deserialize(stream);
   ```
   这里没有显式提供任何对象有关的信息，这些信息是由序列化器通过反射get到的
2. 如果序列化多个对象，反序列化也需要按照序列化时的顺序进行
3. 机制：
   1. 序列化时*类型和程序集的强命名*会被写入流，然后反射得到可序列化的`MemberInfo[]`
   2. 反序列化时*先加载程序集和类型*
   3. 然后*创建类型Uninitialized实例*，并以流中的数据来初始化该实例
4. 如果需要序列化的对象并非都可以序列化，则会在序列化*期间*抛出异常
   1. 在期间抛出异常的原因是，考虑到性能，序列化前CLR不会验证所有的字段、对象都可以序列化
5. 可以应用在
   1. 值类型
   2. 引用类型
   3. 枚举类型
   4. 委托类型
6. Serializable不会继承
### 控制序列化和反序列化
1. `NonSerialized`显式标记某些字段字段不要
3. `OnSerializing`-`OnSerialized`是被序列化对象在序列化前后调用的
   1. 前者用于序列化前准备（欺骗序列化hhh）
   2. 后者用于序列化后恢复之前的准备
4. `OnDeserializing`-`OnDeserialized`
   1. 前者用于反序列化前设置默认值
   2. 后者用于反序列化之后重建待计算值——例如以半径计算面积。和NonSerialized可以协同使用
5. `OptionalField`用于设定某些字段在流中可以有可以没有，没有的话可以不要。我估计这个和OnDeserializing可以一起用
### 格式化器如何序列化类型实例
1. 流程：
   1. `FormatterServerice`反射获取`MemberInfo[]`
   2. `GetObjectData()`传入MemberInfo()获取所有字段的值`Object[]`，和前者是一一对应的
   3. 格式化器将程序集标识和类型完整名称写入流
   4. 依次将两个数组的名字-数据写入流中
2. 反序列化流程：
   1. 将程序集和类型加载到AppDomain
   2. 使用反射调用`GetUninitializedObject()`来分配一个对象的内存但不调用构造器
   3. 使用反射获取MemberInfo[]
   4. 格式化器获取Object[]
   5. PopulateObjectMembers()填充对象
### 控制序列化/反序列化数据
1. 就是自己实现ISerializable接口
   1. 只有一个`GetObjectData()`
   2. 优先级大于`Serializable`特性
2. 要求：
   1. 派生类也必须实现
   2. 必须在构造的时候先调用基类的GetObjectData()
3. 组成部分：
   1. 一个构造器用于存储SerializationInfo（可以用于直接初始化对象的）
      1. 在其中可以调用info的各种GetValue方法
   2. 一个GetObjectData(SerializationInfo info, StreamingContext context)
   3. IDeserializationCallback.OnDeserialization(Object sender)
### 流上下文
1. 表示序列化的数据来源和去向的类型（同一机器？进程？还是别的）
2. 可以在序列化器的`Serialize()`和`Deserialize()`中传入
### 序列化代理
1. 序列化部分和ISerializable基本一致，
   1. 反序列化部分不是构造器了，而是`Object SetObjectData(Object, SerializationInfo, SteamingContext)`
2. 使用的时候需要做两个事：
   1. 创建一个*代理选择器*，告诉他为该种类型使用该种代理
   2. 创建一个*序列化器*，告诉他使用该种代理选择器


### 错题
1. `internal`和`public`是否可以同时存在？
   1. 首先这两者是无法修饰同一个成员的
   2. 但对于类和类的成员可以设定不同的修饰，有两个组合：
      1. A public member of a class or struct is a member that is accessible to anything that can access the containing type. So a public member of an internal class is effectively internal. [StackOverflow](https://stackoverflow.com/questions/9302236/why-use-a-public-method-in-an-internal-class/9302642#:~:text=A%20public%20member%20of%20a,internal%20class%20is%20effectively%20internal.). 即如果类型为`internal`但其中的成员为`public`，那么成员实际也会是`internal`的，或许意义不大。
      2. 但是如果类型为`public`，但成员为`internal`，则该类的可见性在程序集边界上被分成了两块，一块仅供程序集内部访问，一块可以供程序集之外使用。这样是ok的。
2. 以下情况类型对象会不会被构建
   `SomeType p = (SomeType)CreateInstance();`
   1. 如果`CreateInstance`返回为`null`则不会被创建，毕竟引用类型变量都相同。强制类型转换也不涉及对象的初始化和成员调用，所以此时无需创建对象。
3. `in`修饰形参时的作用
   1. 和`ref`、`out`相同，保证参数按引用传递。
   2. 但类似于C++中的`const`引用，`in`保证该参数本身不会在函数内被修改。
4. AIP存在什么问题？
   1. 无法内联初始化。
   2. 由于AIP的field命名由编译器控制，名称可能会更改，所以依赖元数据中的field命名的反序列化机制可能无法正确反序列化该类型实例。
5. 事件和委托之间的差别
   1. 委托是类型，事件是包装了对应委托类型的对象
   2. 事件包装了对应类型的委托：添加了`add`，`remove`的机制，同时取消了外部对于委托的触发权限等。