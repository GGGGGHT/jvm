# class
hotspot为instanceKlass定义了6个状态
```c
enum ClassState {
    allocated,                          // allocated (but not yet linked)
    loaded,                             // loaded and inserted in class hierarchy (but not linked yet)
    linked,                             // successfully linked/verified (but not initialized yet)
    being_initialized,                  // currently running class initializer
    fully_initialized,                  // initialized (successfull final state)
    initialization_error                // error happened during initialization
  };
```
- allocated 已分配，但尚未链接 
- loaded 已加载并已插入JVM内部类层次体系中，尚未链接 
- linked 已链接，尚未初始化
- being_initialized 初始化中
- fully_initialized 完成初始化
- initialization_error 初始化过程中出错

虚拟机规定，一个Java类，首先需要从Class文件中以字节流读取出来，然后你这次经`加载`，`链接`和`初始化`这些逻辑阶段，才会成为JVM能够识别的格式并成为可用状态。
虚拟机规范并规定具体实现必须按照这一顺序进行，hotspot中将一些链接过程细节前移至加载阶段中实现了。例如对Class文件的魔数和版本号信息在加载过程中读取到这部分数据时就
顺便验证了，而不是暂存下来传递给下一阶段（链接）中验证
![image](https://user-images.githubusercontent.com/26846402/130573174-63cefa2d-01ea-4aa6-8e46-7096b6343494.png)

当你这次完成`加载`，`链接`，`初始化`后这个Java类型就可以在JVM中正常被使用了，如创建这个类的实例对象，访问该类的静态域或者调用该类的静态方法



## 加载
ClassFileParser::parseClassFile
## 链接 
InstanceKlass::link_class_impl
## 初始化
