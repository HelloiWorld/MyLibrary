# 读《Objective-C编程之道：iOS设计模式解析》有感

### 写在前面的话
这本书的评价褒贬不一，不过既然买了，过一下才知其好坏


### 关于设计模式
开发中常常有这样的感受：“我以前解决过这个问题，但不太记得具体写法及具体是怎么解决的，记性好一点的时候还需要回头翻看项目源码及注释”，我觉得随着经验的增长，不断熟练API的同时，要有足够多的代码片段积累，更要对重复出现的特定问题有恰当的解决方案，在有需要的时候能够快速、方便的复用到其他场景，充分利用好设计模式，写出更优雅更易于拓展的代码。“我不只是代码的搬运工，我还生产代码。”


# 原型模式
> 使用原型实例指定创建对象的种类，并通过复制这个原型创建新的对象。

脑海中第一想起的就是`NSCopying`协议的`+(id)copyWithZone:(NSZone *)zone`方法，可以显式的对一个对象进行拷贝，获得内存中的资源而不是指针值。常见案例：ViewController之间传递集合类型对象及Model对象时，如果使用了`strong`修饰属性，则只是共享了指针，后续对集合和对象属性的操作都会误操作到前一个ViewController，而对于Model对象使用`copy`修饰则需实现该方法，否则抛出异常。


# 工厂方法模式
> 定义创建对象的接口，让子类决定实例化哪一个类。工厂方法使得一个类的实例化延迟到其子类。

子类只需处理抽象接口，特征由子类对各自生成的对象负责，如果**选择实现**直接在父类就完成了，就退化为“简单工厂模式”。


# 抽象工厂方法模式
> * 抽象工厂：提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。
> * 工厂模式创建一种类型的产品，抽象工厂创建一些列相关的产品家族。

抽象工厂更适合动态切换配置的情况，而且相关接口都有一定关联性，如切换地图（有苹果、百度、高德、谷歌）、切换数据库等，这些都有相似而又不同的方法。


# 生成器模式
> 将一个复杂对象的构建与它的表现分离，使得同样的构建过程可以创建不同的表现。
 
主要目的是分步骤构建复杂对象。将固定不变的构建步骤放到director里独立，变化部分步骤的具体表现放到builder里，这样构建步骤可以拓展和切换。


# 单例模式
> 保证一个类仅有一个实例，并提供一个访问它的全局访问点。

最早你看到的单例写法可能是这样的：

	 + (Singleton *)shareInstance{
         static Singleton * shareInstance = nil;
         @synchronized(self) {
              if (shareInstance == nil) {
          	    shareInstance = [[self alloc] init];
              }
         }
         return shareInstance;
    }
    
这种写法虽然解决了多线程的问题，但仔细阅读代码，发现之后使用单例时每次都会有一次额外的加锁操作，这一部分开销显然是可以避免的，于是结合GCD方法，单例的写法发生了变化：

	static Singleton * shareInstance = nil;
	+ (Singleton *)shareInstance{
    	  static dispatch_once_t onceToken;
    	  dispatch_once(&onceToken, ^{
         		if (shareInstance == nil) {
            		shareInstance = [[self alloc] init];
        		}
    	  });
        return shareInstance;
    }
    
然而这种写法虽然广泛运用，但基于严谨的情况下，如果单例对象被拷贝，这样仍然会出现多个实例，于是我们加上了分配内存区域的方法：

	+ (id)allocWithZone:(struct _NSZone *)zone { 
		static dispatch_once_t onceToken; 
		dispatch_once(&onceToken, ^{ 
			shareInstance = [super allocWithZone:zone]; 
		}); 
		return shareInstance; 
	} 
	
	+ (id)copyWithZone:(struct _NSZone *)zone { 
		return shareInstance; 
	}


# 适配器模式
> 将一个类的接口转换成客户想的另外一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

委托模式。


# 桥接模式
> 将抽象部分与它的实现部分分离，使它们都可以独立地变化。

抽象和实现分离开了，这样它们可以分别独立地扩展，也减少了类的数目。在处理多维度情况时比继承灵活得多。


# 外观模式
> 为系统中的一组接口提供一个统一的接口。外观定义一个高层接口，让子系统更易于使用。

简单一点理解，外观模式就是对单一职责的封装，把高度耦和的事件集中放到一起，外部调用者无需知道具体实现细节，只需唤起统一接口就行。


# 中介者模式
> 用一个对象来封装一系列对象的交互方式。中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。

将对象间多对多的关系转化成一对多的关系，耦合性降低了，但会往往因为渐渐的变得过于庞大而难以维护。


# 观察者模式
> 定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。

经常用到，`NSNotification`和`KVO`


# 组合模式
> 将对象组合成树形结构以表示“部分-整体”的层次结构。组合使得用户对单个对象和组合对象的使用具有一致性。

常用于树形结构，让子节点和叶节点具有一致的行为，不用加以区分。


# 迭代器模式
> 提供一种方法顺序访问一个聚合对象中各个元素，而又不需暴露该对象的内部表示。

常见如`NSEnumertor`抽象类，块枚举及快速枚举。


# 访问者模式
> 表示一个作用于某对象结构中的各元素的操作。它让我们可以在不改变各元素的类的前提下定义作用于这些元素的新操作。

实现复杂，不常用。


# 装饰者模式
> 动态地给一个对象添加一些额外的职责。就拓展功能来说，装饰模式相比生成子类更为灵活。 

比较相像的就是Objective-C的category，动态向类添加行为而无需子类化。category中方法可由子类继承。


# 责任链模式
> 使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间发生耦合。此模式将这些对象连成一条链，并沿着这条链传递请求，直到有一个对象处理它为止。

Cocoa框架中的点击事件就遵循响应责任链模式，如果第一响应者无法处理该事件，就传递到响应链中，交由上层响应者，直到有responder能处理或是传递到`UIApplicationDelegate`。


# 模板方法模式
> 定义一个操作中算法的骨架，而将一些步骤延迟到子类中。模板方法使子类可以重定义算法的某些特定步骤而不改变该算法的结构。

主要目的是定义算法骨架。核心就是把主流程与可变细节分离，父类不关心具体操作细节，子类不必了解父类算法实现步骤。


# 策略模式
> 定义一系列算法，把它们一个个封装起来，并且使它们可相互替换。本模式使得算法可独立于使用它的客户而变化。

常见策略模式应用的一个案例是多个`UITableView`或`UITextField`的判定，在代理中需要对它们的对象做if-else或switch判断，可将相关的代理及数据源方法交由子类或另一个对象持有从而消除判断过程，使逻辑清晰。


# 命令模式
> 将请求封装为一个对象，从而可用不同的请求对客户进行参数化，对请求排队或记录请求日志，以及支持可撤销的操作。

所有的操作看作发出命令，放在有序队列中，这样可以重现以及处理。


# 享元模式
> 运用共享技术有效地支持大量细粒度的对象。

将原本的对象拆分成了享元（内部状态）和外部状态两部分，每个享元会对应一个外部状态。在创建大量相似且不等的对象时，创建一个对象池（对象池的大小取决于对象种类数）保存其内部状态，优先在池中获取对象实例从而节省创建新对象的内存开销。实际上如果外部状态无需存储，而是可以通过计算或传入方式得到，这样才会有优势。


# 代理模式
> 为其他对象提供一种代理以控制对这个对象的访问。

* 延迟加载。在需要时代理对象才将请求转发到真正的实体对象，如网络图片加载前先放置占位图
* 添加权限控制，在请求转发前先验证权限
* 利用`NSProxy`实现消息转发

	    -(void)forwardInvocation:(NSInvocation *)anInvocation;
		-(NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector;


# 备忘录模式
> 在不破坏封装的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态。这样以后就可将该对象恢复到原先保存的状态。

使用如下宏可以利用runtime搞定类序列化

	#define SERIALIZE_CODER_DECODER()     \
	\
	- (id)initWithCoder:(NSCoder *)coder    \
	{   \
	NSLog(@"%s",__func__);  \
	Class cls = [self class];   \
	while (cls != [NSObject class]) {   \
	/*判断是自身类还是父类*/    \
	BOOL bIsSelfClass = (cls == [self class]);  \
	unsigned int iVarCount = 0; \
	unsigned int propVarCount = 0;  \
	unsigned int sharedVarCount = 0;    \
	Ivar *ivarList = bIsSelfClass ? class_copyIvarList([cls class], &iVarCount) : NULL;/*变量列表，含属性以及私有变量*/   \
	objc_property_t *propList = bIsSelfClass ? NULL : class_copyPropertyList(cls, &propVarCount);/*属性列表*/   \
	sharedVarCount = bIsSelfClass ? iVarCount : propVarCount;   \
	\
	for (int i = 0; i < sharedVarCount; i++) {  \
	const char *varName = bIsSelfClass ? ivar_getName(*(ivarList + i)) : property_getName(*(propList + i)); \
	NSString *key = [NSString stringWithUTF8String:varName];   \
	id varValue = [coder decodeObjectForKey:key];   \
	NSArray *filters = @[@"superclass", @"description", @"debugDescription", @"hash"]; \
	if (varValue && [filters containsObject:key] == NO) { \
	[self setValue:varValue forKey:key];    \
	}   \
	}   \
	free(ivarList); \
	free(propList); \
	cls = class_getSuperclass(cls); \
	}   \
	return self;    \
	}   \
	\
	- (void)encodeWithCoder:(NSCoder *)coder    \
	{   \
	NSLog(@"%s",__func__);  \
	Class cls = [self class];   \
	while (cls != [NSObject class]) {   \
	/*判断是自身类还是父类*/    \
	BOOL bIsSelfClass = (cls == [self class]);  \
	unsigned int iVarCount = 0; \
	unsigned int propVarCount = 0;  \
	unsigned int sharedVarCount = 0;    \
	Ivar *ivarList = bIsSelfClass ? class_copyIvarList([cls class], &iVarCount) : NULL;/*变量列表，含属性以及私有变量*/   \
	objc_property_t *propList = bIsSelfClass ? NULL : class_copyPropertyList(cls, &propVarCount);/*属性列表*/ \
	sharedVarCount = bIsSelfClass ? iVarCount : propVarCount;   \
	\
	for (int i = 0; i < sharedVarCount; i++) {  \
	const char *varName = bIsSelfClass ? ivar_getName(*(ivarList + i)) : property_getName(*(propList + i)); \
	NSString *key = [NSString stringWithUTF8String:varName];    \
	/*valueForKey只能获取本类所有变量以及所有层级父类的属性，不包含任何父类的私有变量(会崩溃)*/  \
	id varValue = [self valueForKey:key];   \
	NSArray *filters = @[@"superclass", @"description", @"debugDescription", @"hash"]; \
	if (varValue && [filters containsObject:key] == NO) { \
	[coder encodeObject:varValue forKey:key];   \
	}   \
	}   \
	free(ivarList); \
	free(propList); \
	cls = class_getSuperclass(cls); \
	}   \
	}


### 写在最后的话
粗看完之后，对于很多不常用设计模式使用时机仍然不怎么清楚，[黑人问号脸]❓❓❓