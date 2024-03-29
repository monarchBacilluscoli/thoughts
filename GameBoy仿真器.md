# GameBoy仿真器

http://accu.cc/content/gameboy/preface/

"硬件仿真器"是虚拟机的一个子类

硬件仿真器来指代一种在通用计算机平台上模拟领域专用物理元器件的模拟程序. 

## Python 是解释器还是虚拟机（JAVA为例）
> 在这里可能需要花比较大的篇幅去解释另一个东西: Python 也是将源码 py 编译成字节码 pyc 再执行, 那么为什么这个执行工具被称为 Python 解释器而不是 Python 虚拟机?
> 
> 要解释这个问题, 需要从 CPU 讲起. 虚拟机的原始使命是模拟另一台计算机(它可能是一台真实存在过的计算机, 也可能是一台虚构的计算机), 而"计算"的核心本质是 CPU. CPU 拥有一组特定的指令, 该指令集独立于任何特定的编程语言或使用环境. 指令仅基于 CPU 的当前状态确定性地执行, 并且不依赖于该时间点的指令流中的其他地方的信息. JVM 符合这个描述, 它工作在非常低的层面上, 几乎与 CPU 相同.
> 
> 但对于 Python 解释器来说, 情况就有点不一样了. 它解析语法流, 并且特定时间点的语法 Token 必须依赖上下文 Token 进行解析. 解释器不能孤立地查看每个字节甚至每一行, 并确切知道下一步该做什么. Python 字节码中的字节不能像 JVM 中的字节那样独立运行. 简单来说, Python 的字节码 pyc 只是将计算机程序相对不好解释分析的源码转换成另一种相对好点的源码, 它的本质仍然是解释执行.

懂了，依赖上下文就叫解释器，不依赖上下文就叫虚拟机
但是程序哪怕就是机器码也是依赖于CPU上下文的，那么这里的确切定义应该是——依赖于当前运行时上下文的叫解释器。

> 模拟器最大的特点就是, 代码中没有一行被用作"寄存器级或时钟级的建模", 而主要焦点都放在, 在架构相同的硬件中, 如何在当前的 A 操作系统中, 模拟另一个 B 操作系统运行时的环境.

游戏卡带的内容能直接被映射到系统的标准地址空间

指令类型：
1. 数据存储指令
2. 算数逻辑指令
3. 流程控制