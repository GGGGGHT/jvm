# interpreter

![image](https://user-images.githubusercontent.com/26846402/131253871-23119457-ed98-42e2-9fd6-fec1298410a1.png)<br/>

source: [abstractInterpreter](https://github.com/openjdk/jdk/blob/master/src/hotspot/share/interpreter/abstractInterpreter.hpp)
AbstractInterpreter定义了基于汇编模型的解释器以及解释器生成器的抽象行为,包含一些与平台无关的公共操作 <br/>
在hotspot中实现了两种具体的解释器,即`模板解释器`和`C++`解释器,它们分别由`TemplateInterpreter`子模块和`CppInterpreter`子模块实现.
`模板解释器`是hotspot默认的解释器
