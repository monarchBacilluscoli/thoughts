# Win32多线程程序设计

1. 调用约定是什么？（31）
2. 前台线程和后台线程的差别是什么？
	Backgroung threads are identical to the foreground threads with one exception: a background thread does not keep the process running if all foreground threads have terminated. 
	1.什么情况下创建的是前台什么时候是后台？
		* 前台：
			1. Main application thread
			2. by calling Thread constructor.
		* 后台：
			1. Thread Pool threads.
			2. All threads that enter the managed execution environment from unmanaged code.
3. 内核对象是什么？如何理解内核态，如何理解用户态？
	“用户态”和“内核态”是因为有多用户，多任务的需求，然后在CPU硬件厂商配合之后，产生的一个“操作系统解决多用户多任务需求的方案”。
	这个方案的核心（之一），是“限制”。
	通过硬件手段（也只能硬件手段才能做到），限制某些代码，使其无法控制整个物理硬件，进而使各个不同用户，不同任务的代码，无权修改整个物理硬件，再进而保护操作系统的核心底层代码和其他用户的数据不被无意或者有意地破坏和盗取。——https://www.zhihu.com/question/355721858
4. Win32API和windows系统调用的差别是什么？
	http://reader.epubee.com/books/mobile/97/97b10ad47479941333e5f430bbb37310/text00038.html
	说白了库调用和系统调用并非一一对应关系，毕竟API相对稳定，而系统时时进步。
	现代操作系统：http://reader.epubee.com/books/mobile/97/97b10ad47479941333e5f430bbb37310/text00001.html
5. IOCP到底要解决Overlapped I/O的什么问题？->IOCP到底优势在哪？
	解决了8再回头看这个问题；
	1. overlapped IO核心机制没啥问题，event方式激活也可以让其他线程来负责服务，但wait event次数有上限64（为啥），这就制约了它的上限。自己去扩展也可以但是不方便。
	2. APC就只能由caller来定义callback不够灵活，并且每个请求在后台去开一次线程做任务似乎并不能较好地适应这个系统的机能。再加上还有小请求阻塞进行，效率也不是很高。
	3. IOCP就在灵活性和省事性上有一个比较好的平衡，它给了一个架构，你自己去设定这个架构里各个部分，比如:
		* worker线程数量可以根据机能设定，处理的callback也并非要由caller线程指定，只要GetQueuedCompletionResult，这些worker就和这个框架建立联系了，耦合比较松，扩展性很棒。
		* 然后框架给你提供了由CreateCP函数时指定的ContexKey这一完全自定的对象，加上IO specified overlapp对象，来进行caller到worker的上下文信息传递，不像自己等待event的机制还需要自己维护event->context的表或者采取什么别的传递context方式方才能够进行不同的特定操作；或者WaitOverlappedResult的方式通过自行维护传递handle，Overlap以及别的信息来进行处理（关键这个Wait函数只能Wait一个，这岂不是又进入了一个IO一个线程的怪圈）。
		* 然后文件和IOCP绑定之后所有该文件的IO都由IOCP包办也是很利落的。
		总之就是耦合比较松，然后包办的部分省了事。但是框架的逻辑也是需要一点时间去掌握的上手比较费劲。
	注意几点：
		1. port是和文件、端口的handle绑定的
		2. ContextKey是在port定义的时候给的
6. Overlapped结构的作用？
	153页解释的真他妈差，它是一个概观，就是来自一个已然了解了这个细节的人的总结，然而这个概观并不适合在提纲挈领的位置抛出。
	回放整个overlapped IO过程以及细节我们有几个点需要注意：
	1. 我们给到了一个读文件请求，这个请求在背后完成，我们需要知道它什么时候完成，这个Check完成与否由GetOverlappedResult调用来check，此处需要提供你是哪一个overlapped IO，也就是那个overlappedIO参数——“它像一把钥匙...识别Overlapped操作”。
	2. 此外由于异步读取可能会使用多个overlapped IO去读写同一个文件，所以并没有绑定到文件handle的“当前文件位置”这种状态信息，需要另外一个地方来存储/指定（用户要给调用的）；并且“实际读取的字节数地址”那个参数在异步调用中不会被用到，所以需要另一个参数来返回这个数据（调用要返回的）——“它在你和系统之间提供了一个共享区域...双向传递”
	此外还有一个Event类型参数，这个参数实际上是和wait...系列函数协作去处理单个文件handle的多个IO操作的等待。因为虽然每个操作对应一个OVERLAPPED变量，但是并没有OVERLAPPED数组为参数的GetOverlappedResults之类的函数。所以只好再加一个event...真特么难受...
7. GetOverlappedResult的作用是什么？
	你可以用Wait...系列函数去wait那个文件handle。但这个对象被激活只能表明这个操作已经完成而并不一定成功。题中函数即可确证这里的结果如何。
8. Overlapped IO的整体概观。
	构建一个异步的信息交换结构->发起一个异步请求->等待成功并check确切结果。我并非只能由caller线程来等待这个结果，毕竟有event和handle还有overlapped结构，我完全可以交付给另一个线程——但是等待有上限。并且回调表要自行维护。
	所以才有了APC。
9. APC的整体概观。
	就是回调函数么。
	从OverlappedIO到APC的演进还蛮符合人类越来越懒惰的思路的。
10. 8086通用寄存器有四个AX/BX/CX/DX，可以分为高位和低位H/L
	esp和ebp分别是栈顶的顶指针和底指针
11. 读写锁我的思路
	加入写优先，那么要求就是：
		1. 如果一个写请求进来别的任何读请求和写请求都不能进来->写请求是互斥的，简单用一个mutex就行
		2. 如果有写请求在等待，那么读请求不能进新的->读请求要等一个flag或者count到0（因为有多个写请求）
		3. 如果读请求还没终止，写请求不能进入->有几个读请求的count，进去++，出来--
		所有count或者flag都是atomic的
	我不知道我咋突然想到了另一个思路，并且发现之前的思路实际上是几层混在一起了：
		1. 上层：实际很简单，m个读n个写，实际上可以看作是n+1个事互斥：n就是那n个写，1就是所有的读。
			然后这按惯例肯定得有一个mutex，姑且叫bigMtx
		2. 中层：写进程就等bigMtx就行，结构很简单。但是所有的读线程要变成一个整体，实现“有任意线程在读就得上bigMtx”——所以需要一个特定实现
		3. 下层：咋实现呢，所有读线程肯定要有共享状态才能互相知道什么时候“我是最后一个，可以unlock了”/“我是第一个应该lock上”这个逻辑。后者简单就是每个读线程等锁么，等到就lock，好说；但前者，就是最后一个退出者需要unlock，那无非就是一个count，count为0就unlock表示我们这伙读线程ojbk了你写吧，挺好。
		4. 下下层：具体这个count是一个共享状态，大家都在读写，并且我们希望count和的读写和锁住bigMtx这一组操作是原子的——那就再上一个mutex保护住这俩，这个锁就叫smallMtx。
		5. 至于到底偏向谁，就比如一个老板和下级，无非就是谁说谁听。放在程序上，那就是一个flag，谁写谁读。写的那一方就是被偏向的。
			具体点说就是，比如偏向读，那么就是如果有进程要读了，它就设定flag为true，然后写进程就要先check这个flag为false（得到允许了）才能lock bigMutex并且写。
			反之则相反。实际上就是再提高一层，把读和写当作两个整体。
			后来想想也不是，flag作用太贫弱怕是搞不定，而有count就相当于偏向读了，count+bigMutex就起到了Flag的作用；从读到写才需要有一个“正在等待的写进程数量”的东西来通知读不要再进了。
		6. 具体怎么实现，那么各展其能吧。
	需要注意C# Mutex（包括win32的mutex, c++ std的mutex）不能作为bigMutex的实现，原因是*调用Mutex.ReleaseMutex()的必须是acquire这个mutex的thread*，它有“属于某个thread”的特性，导致最后一个走的不一定能release第一个mutex，而1-Semaphore则没有。看定义mutex就是to protect shared data from being simultaneously accessed by multiple threads. 言下之意就是只能一个thread访问
	此外，count自己读写不需要什么锁，32位的CPU的32位以内简单读写应该是原子的。
12. 为什么存在着size_t, LPCSTR, wchar_t等一些落实到底是unsigned int的别名？
	如果它只是一个名对应一个别名这种写死的对应关系那有啥意义呢？
	好，针对第二个问题，这并不是一个一个的写死的对应关系，关键是注意到有条件编译这个玩意。比如size_t在不同的平台上它的实际type可能就不一样，16位可能只有16位长度，32位可能有32位长度。
	所以typedef至少可以做两件事：1. def一个有意义能看得懂的名字 2. 将跨平台的不同底层实现抽象为同一种类型。
13. 我也不知道看那么多_beginthread, beginthreadex干啥，也用不上，还有一种吃了屎的感觉。
	https://stackoverflow.com/questions/331536/windows-threading-beginthread-vs-beginthreadex-vs-createthread-c). （这里解释了这一坨屎是什么）
14. 函数调用约定的作用？
	从汇编的函数执行+栈准备函数的角度去理解，因为CPU执行起函数来（汇编）只是从栈里一个个pop出参数，参数的压栈顺序非常重要，否则就错了。
	https://www.laruence.com/2008/04/01/116.html
15. 补一补汇编语言吧
	几个寄存器的作用，一些符号的意义。
	地址后面的h表示什么意思？Hex，16位表示。
16. 内存分段和分页指的是什么？
	为了隔离，不是有虚拟地址么，需要映射到实际内存地址。计算出程序需要多少内存比如100M，那就在内存上找100M连续空间映射该虚拟内存到上面。整个程序换入换出。段寄存器保存段号，要去段表里面找段（段号->段基地址，段界限）。
	后来发现分段粒度太大，整体换入换出效率太低，于是把整段程序拆成页，需要页呢，就加载那一页程序到内存里。用页表来维护这个映射关系。
	1. 页表保存在哪里？MMU
	2. 换入/换出是什么？需要页的时候从硬盘加载/暂时不需要页需要释放一些空间把页扔到硬盘。
	？那岂不是分段已经被弃用了？哦，17问链接里面有把虚拟地址分成四个段的，分别是栈堆代码和数据。哦！大颗粒用分段，小颗粒用分页。段表保存页表的地址。当然现在的CPU好像都不一定用分段了。
	3. 至于后来分级页表，比如管理4gb空间需要4mb的页表，页表如果1024个（一级）1024大小（二级）的页表，一级和二级每项都是4字节大小（32位），那就需要4k+4k*1k>4M（不分级的页表大小），反而空间占用更大？
		当然，但是要注意一个问题就是程序运行的“局部化”，就是首先4gb内存肯定是用不完；其次一般也不是所有的页都要在内存上；并且需要的页一般都是连着的->所以我们可以完全不加载没有用到的二级页表！比如如果只有20%页表被加载了，实际占用就只有4k+4M*0.2=0.804M，很节约了对把！这就体现了分级的优势了！
	4. 缺页异常指的是什么？就是程序执行到需要那个页表了但是没在内存里就会触发异常，权限交给操作系统然后补上那一页。
17. CS:EIP的意义，看
	https://zhuanlan.zhihu.com/p/152119007
18. 汇编语言的一些符号意义（不懂函数调用约定中入栈顺序等有何影响特此学习）
	一些括号，$，%什么其实都跟寻址有关系
	当然有一些符号是有差别的，Intel和AT&T规范不同，以AT&T为准吧
	1. mov指令与寻址方式：
		* 寄存器寻址：movl %eax, %edx ;将eax赋值给edx
		* 立即寻址：movl $0x123 %edx ;存入立即数
		* 直接寻址：movl 0x123 %edx ;0x123代表内存地址，将该地址的内容存入edx中。
		* 间接寻址：movl (%ebx), %edx ;将前者存储的内容作为地址，将该地址处的内容存入edx
		* 变址寻址：movl 4(%ebx), %edx ;将前者存储的内容作为地址，加上偏移量4，将该地址的内容存入edx之中。
	2. 一些指令：
		* movl都知道了
		* lea
		* `push` 该操作*先减少esp的值*，然后将源操作数复制到堆栈
		* `lea` load effective address：获取有效地址，给的值是地址值，而不是存储值
		* and：对两个数进行按位与操作，并将结果放在目标操作数中
		* `call` 做两个事，第一个将当前的IP或者cs和ip压入到栈中，并转移。(ebp的存储不借助这玩意干，那应该是call进入函数之后下一个栈帧了)
		* `leave`和`ret`两个指令负责返回，前者负责ebp恢复到上个栈帧ebp，后者负责eip恢复到caller的地址。（这两者都在当前ebp高地址方向，紧挨着ebp）
	3. 一些寄存器相关：
		* eax一般用于保存返回值，所以main函数return 0对应的是movl $0x0 %eax
		* 一般进入函数调用的时候会先保存之前caller的栈帧，通过两个命令：
			push %ebp ;保存了之前的栈帧底(注意这会儿esp也增加，因为push命令有这个效果)
			movl %esp, %ebp ;将栈帧底设定为当前的栈帧顶（要开新的栈了）
			https://www.cnblogs.com/xiaojianliu/articles/8733560.html
		* 函数执行的时候ebp不动，所有的变量和参数都通过offset(%ebp)来搞定。
		* eip保存当前指令的地址，每执行完一条就+1
		* esp保存当前栈顶么，每次push或者pop的时候都会相应地有变e
	4. 所以函数调用以汇编的视角是怎样的过程：
		1. caller将esp前进合理的数值用于放置局部变量，之后esp指向局部变量的栈顶。之后进行各种操作
		2. caller call callee，这个指令会将当前eip压入到栈中（保存当前层级的*指令状态*），并跳转到collee的指令地址（注意此时esp前进）
		3. ——————————下面就是callee——————————
		3. callee将当前ebp（实际上还是caller的ebp）压栈（保存前一层级的*栈帧底状态*，实际就是内存的分布状态，因为这个基址会被用于访问各种局部变量）
		4，callee将esp的赋值给ebp，实际就是更新当前栈帧的栈底。
		5. 回到1的动作，构建callee的局部变量，实际上是通过ebp后退到某些offset取道上一级的实参，这些偏移应该是由编译器来进行计算的。
		猜测不同的调用约定下，比如
19. 数组名和指针的差别
	数组名是个label，而一般使用的时候常用被转换为&arr[0]所以就容易被认为它是指针，但实际上不是。
	有两种情况不会被转换为指针，比如
	`sizeof(arr)`，这个居然会直接返回数组的容量！你猜为啥？因为这玩意在编译的时候就已经被求值过了，是用实际的值替换的这个表达式。
	这也就是为啥c的数组非要静态表达式大小的尺寸了。
	https://blog.csdn.net/u014801596/article/details/79141493
20. 那些lp、sz啥的都是啥玩意
	匈牙利命名法 属性+类型+名称。
	* 属性：g全局，m成员，l仅模块内使用。
	* 类型：sz实际上是字符串的意思
	这玩意是微软内部的某个程序员的习惯，然后传开了
21. CloseHandle到底有啥用？
	用于表示我这个地方（应该是线程？）不会再用这个HANDLE指向的那个系统管理的内核对象了。实际上会让该内核对象的ref count --，然后到0了这个就可能被系统销毁。
22. Dispose()和Finalize()的异同？using的用法？(发现和c++一混，就有点迷了)
23. catch和finally块不会回到try块之中，而是直接往后去了。

## 以下开始CLR via C#
## Thread Basics
24. 线程结构都包含什么？有什么用处？
25. 上下文切换的代价体现在cache上的是什么？
26. CallContext都是啥？Execution Context都包括啥？
27. 什么是LINQ，就是C#提供的一组一致的带有强类型辅助的，可以实现filtering, ordering, and grouping operations on data sources with a minimum of code。这个东西可以用于SQL Server databases, XML documents, ADO.NET Datasets, and any collection of objects that supports IEnumerable or the generic IEnumerable<T> interface，以及一些第三方提供的玩意。
	https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/
	它其实就是在IEnumerable<>基础上添加的一组extension methods。比如IEnumerable.Where()(filter), IEnumerable.Select()（把结果重建为另一种类型，比如提取Student之中的个人信息里的Name和Age）之类。
	let就是在一个query语句之中的局部变量，用于保存中间结果啊，保存一些设置啊啥的。说白了就是个局部变量，可能是因为LINQ的语法和c#标准语法没法完全协调所以才有了这个玩意。
28. 为什么`QLINQ`的query结果给Parallel.ForEach就不能节省时间嘞？而Array却可以
29. 整个状态机的构建涉及到几个关键问题：
	1. 首先如何处理函数本身异步加上内部还有异步调用？就是如何处理这种“部署->做别的事->返回”的流程逻辑。
		这里显然是使用了一个可重入的单一函数形式（主要）的状态机（当然状态机是需要状态的，整体上是一个对象），状态机可以解决“当什么什么时候->做什么什么”
	2. 主要使用可重入的单一函数实现转移，需要解决什么问题？
		1. 记录当前状态，以便下一次根据当前状态继续。
		2. 有状态转移逻辑，根据当前状态如何转移到下一个状态。
		3. 函数需要有外部的“set aside->事件触发->通知并进行下一步”支持，这个机制无法仅靠函数自身逻辑解决。
		对应前三者，这里有一些手段：
		1. 状态有一个index，此外所有临时变量都在一个对象之中保存，包括可以获知函数内部异步调用状态的awaiter
		2. switch根据状态的index和各种状态进行判断转移
		3. set aside->事件->通知返回 机制由函数内部对XXXOnCompleted()函数进行调用安排给CLR解决（?尚未明确m_builder的作用）。set aside的等待机制由return在每一次部署命令后直接返回caller，直接避免在函数内部忙等/死等交出时间片等context switch。
		此外这里还有一些tricks：
		1. for循环的实现使用了拆分的for语句+go to实现，与状态判断结合，保证了除了第一次进入之外后续不会进入prolog，以及完成ForBody中异步任务后下一次重入必然进入Epilog的机制。
		2. 外部try直接包了所有的代码（内部业务try那就是另一回事了），catch到了之后给了builder保存以便以后处理（这里也反映了一般异步调用的异常处理机制——先存住，等你要结果的时候再扔给你）。
		3. ...
	可以看出这个玩意的设计还是相当需要一些知识和相关的素养的。
	剩余还有一些问题：
	1. Builder的作用
	2. awaiter的作用
30. `await`操作符的作用：The `await` operator suspends evaluation of the enclosing **async method** until the asynchronous operation represented by its operand completes. When the asynchronous operation completes, the await operator returns the result of the operation, if any. When the await operator is applied to the operand that represents an already completed operation, it returns the result of the operation immediately without suspension of the enclosing method. The await operator doesn't block the thread that evaluates the async method. **When the await operator suspends the enclosing async method, the control returns to the caller of the method**.
虽然没有带例子，但是这一段描述的已经非常棒了：
In the following example, the HttpClient.GetByteArrayAsync method returns the `Task<byte[]>` instance, which represents an asynchronous operation that produces a byte array when it completes. Until the operation completes, the await operator suspends the DownloadDocsMainPageAsync method. **When DownloadDocsMainPageAsync gets suspended, control is returned to the Main method**, which is the caller of DownloadDocsMainPageAsync. **The Main method executes until it needs the result of the asynchronous operation performed by the DownloadDocsMainPageAsync method**. When GetByteArrayAsync gets all the bytes, the rest of the DownloadDocsMainPageAsync method is evaluated. After that, the rest of the Main method is evaluated.
https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/await


# 段瑞提示
1. 最重要的部分是IOCP，最重要的底层异步操作都是基于IOCP完成的。
2. windows核心编程第5版8、9、10章。
3. IOCP这一部分主要是用于在Windows上面实现异步IO，可以结合CLR644-645面去看。
4. 26章之后的源码都推荐看一下。之后就可以多写一些例子，socket之类的。
5. lock 和邻接去由什么关系
6. （好象是704页）同步和异步构造，异步锁的实现，服务器的框架和程序。
7. 服务器基本上就是照着这本书来写的，服务器程序员需要把这本书精通。


