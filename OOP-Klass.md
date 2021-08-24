# OOP-Klass模型

- OOP ordinary object pointer,或OOPS,即普通对象指针,用来描述对象实例信息
- Klass: Java类的C++对等体,用来描述Java类

对于OOPS对象来说,主要职能在于表示对象的实体数据,没必要持有任何虚函数,而在描述Java类的Klass对象中含有VTBL(继承自Klass父类Klass_vtbl),那么,Klass对象能够根据Java对象的实际类型
进行C++的分发,这样一来,OOPS对象只需要通过相应的Klass便可以找到所有的虚函数,避免了在每个对象中都分配一个C++ VTBL指针. <br/>

Klass向JVM提供两个功能:
  - 实现语言层面的Java类
  - 实现Java对象的分发功能.
