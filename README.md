# Objective-C
Objective-C语法和底层知识的整理，方便个人复习。
# Objective-C本质
Objective-C的底层代码其实都是由C/C++来实现的，Objective-C中的对象就有C++中的结构体这一种数据结构来构造。<br>一个NSObject对象，系统分配了16个字节给NSObject对象（通过malloc_size函数获得），但NSObject对象内部只使用了8个字节的空间（64bit环境下，可以通过class_getInstanceSize函数获得）。这里设计到一个知识点：内存对齐。<br> 
* 内存对齐：
# Objective-C对象的分类
OC对象有3种：实例对象、类对象和元类对象。
## 实例对象(instance对象)
* instance对象就是通过类alloc出来的对象，每次调用alloc都会产生新的instance对象<br>
```
NSObject *object1 = [[NSObject alloc] init];
NSObject *object2 = [[NSObject alloc] init];
```
object1和object2分别是两个不同的实例对象，分别占据两块不同的内存空间。<br>实例对象在内存中的存储信息包括：isa指针和成员变量的具体值。
## 类对象（Class对象）
* 
