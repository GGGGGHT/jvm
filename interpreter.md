# interpreter

![image](https://user-images.githubusercontent.com/26846402/131253871-23119457-ed98-42e2-9fd6-fec1298410a1.png)<br/>

source: [abstractInterpreter](https://github.com/openjdk/jdk/blob/master/src/hotspot/share/interpreter/abstractInterpreter.hpp)
AbstractInterpreter定义了基于汇编模型的解释器以及解释器生成器的抽象行为,包含一些与平台无关的公共操作 <br/>
在hotspot中实现了两种具体的解释器,即`模板解释器`和`C++`解释器,它们分别由`TemplateInterpreter`子模块和`CppInterpreter`子模块实现.
`模板解释器`是hotspot默认的解释器 <br/>

在HotSpot启动时,初始化解释器模块时,会使用解释器生成器一次性生成所有的Codelets.模板解释器使用的生成器为C++类TemplateInterpreterGenerator,通过generate_all函数生成许多虚拟机内部公用全称和字节码codelets

## 模板表与转发表
模板用来描述指定字节码的机器码生成模板的属性,并拥有一个生成器函数用来生成模板.所有的字节码的模板组合在一起,构成一个模板表.表中每个元素都是一个Template,元素按照字节码值的递增顺序排列,第n号元素表示的就是bytecode值为n的Template
