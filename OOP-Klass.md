# OOP-Klass模型

- OOP ordinary object pointer,或OOPS,即普通对象指针,用来描述对象实例信息
- Klass: Java类的C++对等体,用来描述Java类

对于OOPS对象来说,主要职能在于表示对象的实体数据,没必要持有任何虚函数,而在描述Java类的Klass对象中含有VTBL(继承自Klass父类Klass_vtbl),那么,Klass对象能够根据Java对象的实际类型
进行C++的分发,这样一来,OOPS对象只需要通过相应的Klass便可以找到所有的虚函数,避免了在每个对象中都分配一个C++ VTBL指针. <br/>

Klass向JVM提供两个功能:
  - 实现语言层面的Java类
  - 实现Java对象的分发功能.

|模块|模块功能|
|--|--|
|oop|定义了在OOPS共同基类|
|constantPoolOop|表示在Class文件中描述的常量池|
|cpCacheOop|即constantPoolCacheOop 是与constantPoolOop相们生的数据结构,缓存了字段和方法的访问信息,为运行时环境快速访问字段和方法提供重要作用|
|arrayOop|定义了数组OOPS的抽象基类|
|instanceOop|表示一个Java类型实例|
|makrOop|表示对象头|
|typeArrayOop|表示容纳基本类型的数组|
|constMethodOop|表示一个Java方法中的不变信息|
|methodDataOop|记录恨不能信息的数据结构|
|methodOop|表示一个Java方法|
|objArrayOop|表示一个持有OOPS的数组|
|klassOop|描述一个与Java类对等的C++类|
|oopsHierachy|描述了对象表示层次|
|Klass|klassOop的一部分,用来描述语言层的类型|
|instanceKlass|虚拟机层面描述一个java类|

![image](https://user-images.githubusercontent.com/26846402/130542414-0cec51a2-b741-4998-8283-c612735396a9.png)

在虚拟机内部通过instanceOopDesc来表示一个Java对象. 对象在内存中的而已可以分为连续的两部分: instanceOopDesc和实例数据
```c
volatile markOop  _mark;
  union _metadata {
    Klass*      _klass;
    narrowKlass _compressed_klass;
  } _metadata;
```
instanceOopDesc或arrayOopDesc又被称为对象头,对象头包括两部分信息:
- _mark 存储对象运行时记录信息,如hashcode GC分代年龄 线程持有的锁,偏向线程id..
- metadata指针 指向描述类型的Klass对象的指针,Klass对象包含了实例对象所属类型的元数据. 虚拟机频繁使用这个指针定位到位于方法区内的类型信息.

arrayOopDesc与instanceOopDesc都拥有继承自基类的oopDesc的mark word和元数据指针.但二者在对象头上的唯一区别在于,arrayOop增加一个描述数组长度的字段

在Java应用程序运行过程中，每创建一个Java对象，在JVM内部也相应创建一个对象头，因此对象头的内存布局设计关乎着内存空间的利用率。
OOP框架中采用一项内存优化措施是对类元数据指针进行压缩存储
-XX：+UseCompressedOops 在64位JVM上，对类元数据指针(_metadata成员) 使用32位指针存储
开启指针压缩能在一定程度上降低开销

实例字段存储的顺序： 
按照longs/doubles ints shorts/chars bytes/booleans OOPS的顺序进行分配。相同宽度的字段问题被分配到一起。在满足这个前提条件的前提下，可能会出现
一种情况即在父类中定义的变量可能会出现在子类之前。在默认情况下，VM选项`CompactFields`为true，表示子类之中较窄的变量可能会插入到父类变量的空隙之中

