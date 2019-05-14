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
```
Class objectClass1 = [object1 class];
Class objectClass2 = [object2 class];
Class objectClass3 = object_getClass(object1);
Class objectClass4 = object_getClass(object2);
Class objectClass5 = [NSObject class];
```
objectClass1 ~ objectClass5都是NSObject的class对象（类对象）。它们是同一个对象。每个类在内存中有且只有一个class对象。<br>
* 类对象在内存中的存储信息包括：
* isa指针
* superclass指针
* 类的属性信息（@property）、类的对象方法信息（instance method）
* 类的协议信息（protocol）、类的成员变量信息（ivar）
## 元类对象（meta-class对象）
```
Class objectMetaClass = object_getClass(objectClass5);  //Runtime API
// 注意以下方法获得的是类对象而不是元类对象
Class object = [[NSObject class] class];
```
objectMetaClass是NSObject的meta-class对象（元类对象），每个类在内存中有且只有一个meta-class对象。<br>
* 元类对象在内存中的存储信息包括：
* isa指针
* superclass指针
* 类的类方法信息（class mehtod）
## isa指针和superclass指针
### isa指针
![image](https://github.com/lin450922/Objective-C/blob/master/images/isa指针指向.png)













