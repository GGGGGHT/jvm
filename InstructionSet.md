# 指令集结构 
- 寄存器指令集: 有利于代码重生成，数据访问快，非常得优化程序的运行速度，但是操作数需要命名和显示的指定，因此指令较长
- 堆栈型指令集: 简洁直观，指令短小，易于实现 缺点是不能随机访问堆栈元素，不利于代码优化
- 累加型指令集:  机器内部状态较为简单，指令也很短小，累加型复用累加器作为唯一的存储单元。 存储器存取开销最大

HotSpot中的JVM的指令集就是一款堆栈型指令集。<br/>
在一条指令的执行周期内需要经过以下几个周期:
1. 取指(Instruction Fetch)
2. 指令译码(Instruction Decode)
3. 取操作数(Operatand Fetch)
4. 执行(Execute)
5. 存储结果(Result Store)
6. 获取下一条指令(Next Instruction)

## 数据传送指令
### 局部变量 常量池和操作数栈之间的数据传送
用于在虚拟机局部变量和操作数栈之间传送数据的指令主要有3类
- Load类指令(数据方向: 局部变量 -> 操作数栈) 包括`iload_<n>`,`lload_<n>`,`fload_<n>`,`dload_<n>`,`aload_<n>`
- Store类指令(数据方向: 操作数栈 -> 局部变量) 包括`istore_<n>`. etc..
- 能够将来自立即数或常量池的数据传送到操作数栈的指令 包括`bipush`,`sipush`,`ldc` etc..

## 类型转换 
类型转换指令是将两种JVM数值类型相互转换，这些操作一般用于实现用户代码的显式类型转换操作。类型转换方式有两种:
1. 宽化类型转换指令 将小范围类型数据向大范围类型数据转换 
  - int类型到long,float,double 
  - long类型到float,double
  - float到double
  这类指令不会千万数据精度丢失
2. 窄化类型转换指令
  - int类型到byte,short,char
  - long类型到int
  - float到int,long
  - double到int,long,float
