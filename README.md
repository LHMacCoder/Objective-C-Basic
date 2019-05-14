# Objective-C
Objective-C语法和底层知识的整理，方便个人复习。以下所有内容参考自小码哥底层班，如有侵权请联系（qq:294161255）删除。
# Objective-C本质
Objective-C的底层代码其实都是由C/C++来实现的，Objective-C中的对象就有C++中的结构体这一种数据结构来构造。<br>一个NSObject对象，系统分配了16个字节给NSObject对象（通过malloc_size函数获得），但NSObject对象内部只使用了8个字节的空间（64bit环境下，可以通过class_getInstanceSize函数获得）。这里涉及到一个知识点：内存对齐。<br> 
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
# isa指针和superclass指针
## isa指针
![image](https://github.com/lin450922/Objective-C/blob/master/images/isa指针指向.png)
* 实例对象的isa指针指向类对象
 * 当调用对象方法时，通过instance的isa找到class，最后找到对象方法的实现进行调用
* 类对象的isa指针指向元类对象
 * 当调用类方法时，通过class的isa找到meta-class，最后找到类方法的实现进行调用
* 元类对象的isa指向基类的元类对象
## superclass指针
![image](https://github.com/lin450922/Objective-C/blob/master/images/class对象的superclas指针.png)
* 当Student的instance对象要调用Person的对象方法时，会先通过isa找到Student的class，然后通过superclass找到Person的class，最后找到对象方法的实现进行调用
![image](https://github.com/lin450922/Objective-C/blob/master/images/meta-class对象的superclas指针.png)
* 当Student的class要调用Person的类方法时，会先通过isa找到Student的meta-class，然后通过superclass找到Person的meta-class，最后找到类方法的实现进行调用
## isa和superclass总结
![image](https://github.com/lin450922/Objective-C/blob/master/images/isa和superclass.png)
* instance的isa指向class
* class的isa指向meta-class
* meta-class的isa指向基类的meta-class
* class的superclass指向父类的class
  * 如果没有父类，superclass指针为nil
* meta-class的superclass指向父类的meta-class
* 基类的meta-class的superclass指向基类的class
* instance调用对象方法的轨迹
  * isa找到class，方法不存在，就通过superclass找父类
* class调用类方法的轨迹
  * isa找meta-class，方法不存在，就通过superclass找父类
# 类的结构
## isa指针结构
![image](https://github.com/lin450922/Objective-C/blob/master/images/isa指针.png)
* 从64bit开始，isa需要进行一次位运算，才能计算出真实地址。
```
# if __arm64__
#   define ISA_MASK        0x0000000ffffffff8ULL
# elif __x86_64__
#   define ISA_MASK        0x00007ffffffffff8ULL
# endif
```
## 类对象和元类对象的本质
class、meta-class对象的本质结构都是struct objc_class。
### struct objc_class
![image](https://github.com/lin450922/Objective-C/blob/master/images/struct_objc_class.png)
# KVO & KVC
## KVO
KVO的全称是Key-Value Observing，俗称“键值监听”，可以用于监听某个对象属性值的改变。
```
NSKeyValueObservingOptions options = NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld;
[实例对象 addObserver:观察者 forKeyPath:@"key" options:options context:nil];

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context
{
    NSLog(@"监听到%@的%@属性值改变了 - %@ - %@", object, keyPath, change, context);
}
```
### 未使用KVO监听的对象
![image](https://github.com/lin450922/Objective-C/blob/master/images/NO_KVO.png)
可以看到当对象没有被监听的时候，对对象的属性赋值是直接调用了该对象属性的set方法。
### 使用了KVO监听的对象
![image](https://github.com/lin450922/Objective-C/blob/master/images/KVO_Class.png)
当对象被监听之后，Objective-C通过Runtime机制，动态的生成了该对象的一个子类：`NSKVONotifying_类名`，并将原来的实例对象的isa指针指向`NSKVONotifying_类名`这个子类。<br>
当实例对象的属性值被修改后，实例对象通过isa指针找到`NSKVONotifying_类名`这个类对象，然后调用set方法，`NSKVONotifying_类名`重写set方法实现了属性值得监听。
```
- (void)setAge:(int)age
{
    _NSSetIntValueAndNotify();
}
```
\_NSSetIntValueAndNotify()是对整形类型的一个监听实现，除此之外还包括如下类型：
* \_NSSetBoolValueAndNotify()
* \_NSSetCharValueAndNotify()
* \_NSSetDoubleValueAndNotify()
* \_NSSetFloatValueAndNotify()
* \_NSSetLongLongValueAndNotify()
* \_NSSetLongValueAndNotify()
* \_NSSetObjectValueAndNotify()
* \_NSSetPointValueAndNotify()
* \_NSSetRangeValueAndNotify()
* \_NSSetRectValueAndNotify()
* \_NSSetShortValueAndNotify()
* \_NSSetSizeValueAndNotify()
#### \_NSSet*ValueAndNotify的内部实现
```
- (void)setAge:(int)age
{
    _NSSetIntValueAndNotify();
}
// 伪代码
void _NSSetIntValueAndNotify()
{
    [self willChangeValueForKey:@"age"];
    [super setAge:age];
    [self didChangeValueForKey:@"age"];
}
- (void)didChangeValueForKey:(NSString *)key
{
    // 通知监听器，某某属性值发生了改变
    [oberser observeValueForKeyPath:key ofObject:self change:nil context:nil];
}
```
* 调用willChangeValueForKey:
* 调用原来的setter实现
* 调用didChangeValueForKey:
* didChangeValueForKey:内部会调用observer的observeValueForKeyPath:ofObject:change:context:方法
## KVC
KVC的全称是Key-Value Coding，俗称“键值编码”，可以通过一个key来访问某个属性<br>
常见的API有
* \- (void)setValue:(id)value forKeyPath:(NSString *)keyPath;
* \- (void)setValue:(id)value forKey:(NSString *)key;
* \- (id)valueForKeyPath:(NSString *)keyPath;
* \- (id)valueForKey:(NSString *)key; 
### setValue:forKey:原理图
![image](https://github.com/lin450922/Objective-C/blob/master/images/setValueForKey.png)
### valueForKey:原理图
![image](https://github.com/lin450922/Objective-C/blob/master/images/ValueForKey.png)

# Category
## Category的结构
![image](https://github.com/lin450922/Objective-C/blob/master/images/CategoryStruct.png)
## 分类的加载过程
* 通过Runtime加载某个类的所有Category数据
* 把所有Category的方法、属性、协议数据，合并到一个大数组中，后面参与编译的Category数据，会在数组的前面
* 将合并后的分类数据（方法、属性、协议），插入到类原来数据的前面
### +load方法
```
Invoked whenever a class or category is added to the Objective-C runtime; implement this method to perform class-specific behavior upon loading.
```
这是Apple官方文档对于load方法的解释，意思是类和分类的load方法是在Objective-C运行时加载的时候调用的。<br>
每个类、分类的+load，在程序运行过程中只调用一次,而且+load方法是根据方法地址直接调用，并不是经过objc_msgSend函数调用，调用顺序:
* 先调用类的+load
 * 按照编译先后顺序调用（先编译，先调用）
 * 调用子类的+load之前会先调用父类的+load
* 再调用分类的+load
 * 按照编译先后顺序调用（先编译，先调用）
### +initialize方法
```
Initializes the class before it receives its first message.
```
这是Apple官方文档的一段解释，意思是initialize方法是在类第一次接收到消息的时候调用的。<br>
调用顺序
* 先调用父类的+initialize
* 再调用子类的+initialize(先初始化父类，再初始化子类，每个类只会初始化1次)

### +initialize和+load方法对比
+initialize和+load的比较:
* 调用方式
  * load是根据函数地址直接调用
  * initialize是通过objc_msgSend调用，如果子类没有实现+initialize，会调用父类的+initialize（所以父类的+initialize可能会被调用多次）
  * 如果分类实现了+initialize，就覆盖类本身的+initialize调用
* 调用时刻
  * load是runtime加载类、分类的时候调用（只会调用1次）
  * initialize是类第一次接收到消息的时候调用，每一个类只会initialize一次（父类的initialize方法可能会被调用多次）
* load、initialize的调用顺序
  * load
    * 先调用类的load
    * 先编译的类，优先调用load
    * 调用子类的load之前，会先调用父类的load
    * 再调用分类的load
    * 先编译的分类，优先调用load
  * initialize
    * 先初始化父类
    * 再初始化子类（可能最终调用的是父类的initialize方法）
# 关联对象
默认情况下，因为分类底层结构的限制，不能添加成员变量到分类中。但可以通过关联对象来间接实现。<br>
关联对象提供了以下API：
```
// 添加关联对象
void objc_setAssociatedObject(id object, const void * key, id value, objc_AssociationPolicy policy)

// 获得关联对象
id objc_getAssociatedObject(id object, const void * key)

// 移除所有的关联对象
void objc_removeAssociatedObjects(id object)
```
key的常见用法：
```
static void *MyKey = &MyKey;
objc_setAssociatedObject(obj, MyKey, value, OBJC_ASSOCIATION_RETAIN_NONATOMIC)
objc_getAssociatedObject(obj, MyKey)

static char MyKey;
objc_setAssociatedObject(obj, &MyKey, value, OBJC_ASSOCIATION_RETAIN_NONATOMIC)
objc_getAssociatedObject(obj, &MyKey)

// 使用属性名作为key
objc_setAssociatedObject(obj, @"property", value, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
objc_getAssociatedObject(obj, @"property");

// 使用get方法的@selecor作为key
objc_setAssociatedObject(obj, @selector(getter), value, OBJC_ASSOCIATION_RETAIN_NONATOMIC)
objc_getAssociatedObject(obj, @selector(getter))
```
## objc_AssociationPolicy
| objc_AssociationPolicy        | 对应的修饰符  |
| ------------- | -----:|
| OBJC_ASSOCIATION_ASSIGN   | assign |
|  OBJC_ASSOCIATION_RETAIN_NONATOMIC  |  strong,nonatomic |
|  OBJC_ASSOCIATION_COPY_NONATOMIC  |   copy,nonatomic |
|  OBJC_ASSOCIATION_RETAIN  |   strong,atomic |
|  OBJC_ASSOCIATION_COPY  |   copy,atomic |





















