# hotspot源码学习

## hotspot源码结构 
- make
- agent
- src
  - cpu
  - os
  - os_cpu
  - share
    - tools
    - Vm
- test
其中`src`目录下就是HotSpot的主体源码,`share`目录下是独立于操作系统和处理器类型的代码,这部分代码是HotSpot的核心业务,实现了HotSpot的主要功能
`Vm`目录下是实现虚拟机各项功能的`Vm`目录. `Tools`目录下是几个独立的虚拟机工具

Vm模块中的功能
 - adlc               平台描述文件
 - asm                汇编器
 - c1                 client编译器,即C1编译器
 - ci                 动态编译器
 - classfile          class文件解析和类的链接等 
 - code               机器码生成
 - compiler           调用动态编译器接口
 - gc_implementation  垃圾收集器的具体实现
 - gc_interface       GC接口
 - interpreter        解释器
 - libadt             抽象数据结构
 - memory             内存管理
 - oops               JVM内部对象表示
 - opto               C2编译器
 - precompiled        预编译
 - prims              HotSpot对外接口
 - runtime            运行时
 - services           JMX接口
 - shark              基于LLVM实现的即时编译器
 - utilities          内部工具类和公共函数
