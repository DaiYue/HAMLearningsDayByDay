# Effective Objective-C 复习

Q : 我们常说Objective C的消息（messaging）机制，那么Objective C的消息发送和c++的函数调用区别在哪里呢？
  
A : 主要区别就是 c++ 在编译期就会把函数调用转换为具体调用的哪块代码，而 OC 是在运行时才去找的。所以我们才能有那么多 runtime 黑科技

---

Q : SomeClass* someObject = [SomeClass new]; 这行代码的内存分配在什么地方呢？

A : someObject 这个指针在栈区，指向的 object 在堆区啦。如果 someObject 有个 int 类型的属性，这个属性也是放在堆区的

---

Q : 我们知道最好在.m而不是.h里import，为了避免暴露过多细节、防止循环引用，加快编译等。但有哪些情况必须在.h中引用？这些情况下要是循环引用了咋办呢？

A : 必须在 .h 中引用的情形包括继承父类，必须引用父类的 .h；实现 protocol，必须引用 protocol 的 .h。如果造成循环引用了怎么办呢？首先，两个类肯定不能互相继承，父类没有这问题。如果是 protocol，可以把 protocol 单拿出来放到一个 .h 里，跟任何类都不在一起……

---

Q : 常量的命名规范是什么样的？

A : 就是一般情况放在 .m 里，用 k 开头。但是如果要暴露在 .h 里，就要用类名做前缀了，免得重复~

---

Q : 咱们平常定义变量都会static const CGFloat……这样，那不加static可以吗？

A : 加 static const，会在编译优化的时候直接替换，跟 #define 差不多（但比 #define 有很多好处）。加 static 的好处是，可以让这个常量的范围限制在这个 .m 之内。如果不加 static，会在全局创建一个符号，如果有两个重名的就该报错啦。

--- 

Q : 假设我们为一个 enum 写了 switch 语句，switch(someEnum) { case xx: … break; ….} 这种，如果对 enum 的每一个取值都加上了 case 来处理，那么还要写 default 分支吗？为啥？

A : 不写default较好。因为这样，如果后面增加了enum选项，会立即报一个warning，让你不会忘了处理它。一般即使不需要处理的选项，也要把它写出来，也是同样的原因。如果是每个分支直接return的话，最好在函数结尾加一句return。

---

Q : 使用enum的时候，我们经常会把几个选项或起来。那么接收方拿到这个或起来的结果，该怎么使用呢？如何知道里面或了哪几个选项呢？

A : 与上对应的选项 if (options & SomeOption) 即可。唯一要注意的就是不要习惯性地写 == YES。因为与出来的结果可能为二进制的 10、100 等。

---

Q : 你用过@dynamic吗？它有什么用？一般是什么情况下用的？

A : 其实这个 @dynamic 的意思就是跟系统说，不要创建 property 对应的成员变量（就是一般的 _someProperty），也不要自动生成 get/set 方法，同时不要报错，到在运行时我自己会来添加 get/set 方法。比如像 CoreData 的对象，有些属性并不是用 _someProperty 存起来的，而是从数据库里读出来的。那么就不需要系统默认的 getter、setter，而是在运行时生成~

---

Q : `@property (nonatomic, copy) NSMutableArray* array;` 这样写有啥问题~ 就酱=w=

A : 就是 NSMutableArray 一 copy 就会变成一个不可变的 NSArray。所以这个属性的 setter 就会导致，实际存起来的对象是不可变的。一旦调到 NSMutableArray 特有的方法比如 addObject 等，会马上 crash。非常危险。

--- 

Q : 如何重写一个atomic属性的getter，setter呢~大家自己写写看吧：）

A : 就把self锁上就可以了~注意如果你只重写getter或者setter就会报错，因为如果getter或setter不能锁住同一个东西的话就没法实现atomic鸟。

--- 

Q : 在现代的 Objective-C 中，我们经常使用 . 语法来访问和修改属性~ 比如 self.name = @"hamster"; 很少再使用下划线的语法了。但是，请问哪些情况下必须使用下划线语法，而不能使用点语法呢？~

A : 

- 第一种，getter、setter 里必须用下划线语法，原因是显而易见的，不然会无限递归.

- 第二种，init 方法里必须用下划线语法。这个原因好像没有同学说，主要是因为，子类可能会重写 setter 方法。如果 init 里用点语法，父类的 init 方法里会不留神调到子类的 setter，造成意料之外的乌龙，还可能会有危险。

- 第三种，dealloc 里不要用任何点语法~ 一方面还是子类父类的问题，dealloc 是 init 的逆过程，问题同理。另一方面，重写的 getter、setter 里可能会有在 dealloc 阶段来做不安全的行为，要避免触发它们。

---

Q : 我们都知道category 可以给一个类增加新的方法，尤其是系统的类，我们改变不了。那么，可以给一个系统的类增加新的属性吗？category 新加的方法可以使用到这些新的属性吗？

A : 答案是可以和可以。用关联对象的方法。小伙伴提供了几个可以学习的链接：

- [雷纯峰的博客](http://blog.leichunfeng.com/blog/2015/06/26/objective-c-associated-objects-implementation-principle) 主要讲解Associated Objects 的实现原理和内存管理策略

- [sunny的博客](http://blog.sunnyxx.com/2014/03/05/objc_category_secret/) 主要讲解catogry拓展方法的实现原理

- [美团网的博客](http://tech.meituan.com/DiveIntoCategory.html) 更深入讲解category的实现原理

---

Q : 上题所说的关联对象，大家如果看了它的API，会发现跟kvc的key object机制很像。那么，能说一说 associate object 和 kvc 机制的区别在哪里吗？~

A : 这个问题想强调的就是，associated object语法看着像key value，但实际它的key机制跟我们平常习惯的有一点不同，我们习惯的比如NSDictonary，两个key只要isEqual方法返回YES就认为是相同的key，而associated object 的key是类似用==来判断的，不是要求内容相同，而是指针地址必须完全一致，所以我们一般把key存到一个静态变量里~

---

