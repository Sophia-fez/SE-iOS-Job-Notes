# IOS笔试选择、简答题

1. [多线程中栈与堆是公有的还是私有的](https://www.jianshu.com/p/224a308a6347)

   在多线程环境下，每个线程拥有一个栈和一个程序计数器。栈和程序计数器用来保存线程的执行历史和线程的执行状态，是线程私有的资源。其他的资源（比如堆、地址空间、全局变量）是由同一个进程内的多个线程共享。
   
2. [@property 后面可以有哪些修饰符？](https://blog.csdn.net/qq_32744055/article/details/53443805)

   - 线程安全的： atomic, nonatomic
   - 访问权限的： readonly， readwrite
   - 内存管理（ARC） assign，strong，weak，copy
   - 内存管理（MRC）assign， retain，copy
   - 指定方法名称： setter= getter=

3. 多线程中栈与堆是公有的还是私有的

   在多线程环境下，每个线程拥有一个栈和一个程序计数器。栈和程序计数器用来保存线程的执行历史和线程的执行状态，是线程私有的资源。其他的资源（比如堆、地址空间、全局变量）是由同一个进程内的多个线程共享。

4. [UIView的继承链](https://www.jianshu.com/p/78cc7e2627c6)

   NSObject，UIResponder，UIView

5. [frame和bounds的区别](https://www.jianshu.com/p/964313cfbdaa)

   frame: 该view在父view坐标系统中的位置和大小。（参照点是，父亲的坐标系统）
   bounds：该view在本地坐标系统中的位置和大小。（参照点是，本地坐标系统，就相当于ViewB自己的坐标系统，以0,0点为起点）

6. [Quartz2D UIBezierPath()](https://www.jianshu.com/p/963bc1c46dab)

7. [HTTP 304](https://blog.csdn.net/qq_37960324/article/details/83374855)

8. [UIViewController中各方法调用顺序及功能](https://blog.csdn.net/hai_han0716/article/details/44080913)

   loadView, viewDidLoad, viewWillUnload, viewDidUnload, viewWillAppear, viewDidAppear, viewWillLayoutSubviews，viewDidLayoutSubviews，viewWillDisappear, viewDidDisappear

9. [super dealloc]内存释放的先后顺序
   Objective－c 语言中最头疼的事就是内存释放，申明一个变量后记得一定要释放这个变量，相信很多人在dealloc函数[super dealloc]位置这问题上纠结过，经过实践发现，[super dealloc]写在自己释放的内存之前，经常会发生crash，而写在之后不会。对，Objective－c 中不能把自己写的释放内存放在[super dealloc]之后，原因是“你所创建的每个类都是从父类，根类继承来的，有很多实例变量也会继承过来，这部分变量有时候会在你的程序内使用，它们不会自动释放内存，你需要调用父类的 dealloc方法来释放，然而在此之前你需要先把自己所写类中的变量内存先释放掉，否则就会造成你本类中的内存积压，造成泄漏”。当然你使用了iOS6.0之后ARC的话，就不会遇到这个问题啦。

```
(void)dealloc {
[XXX release];

[super dealloc];

......
}
```

**init的过程，感觉是和dealloc相反的**

10. [atomic一定是线程安全的吗](https://www.jianshu.com/p/c40b312153c1),atomic在set方法里加了锁，防止了多线程一直去写这个property，造成难以预计的数值。但这也只是读写的锁定。跟线程安全其实还是差一些

11. [import与#include的区别](https://www.cnblogs.com/hecanlin/articles/9121482.html)

12. [如何让自己的类用 copy 修饰符](https://www.jianshu.com/p/d39270dcd050)，实现NSCopying协议，NSMutableCopying协议

13. [init的过程](https://www.jianshu.com/p/fe8918f5917a)

14. [KVO](https://www.jianshu.com/p/4c68b6f27eae)回调方法：observeValueForKeyPath

15. 需要在手动管理内存分配和释放的Xcode项目中引入和编译用ARC风格编写的文件，需要在文件的Compiler Flags上添加参数： -fobjc-arc，[这篇博客还给出了他的笔试选择题](https://blog.csdn.net/potato512/article/details/43983933)

16. [iOS子线程更新UI的两种方法](https://blog.csdn.net/lvxiangan/article/details/50868558)：performSelectorOnMainThread、dispatch_async(dispatch_get_main_queue(), ^{ ... })

17. [OC如何实现多重继承](https://www.jianshu.com/p/c473b41c083d)

18. [UITableViewDataSource协议](https://www.cnblogs.com/jukaiit/p/4702797.html)

    首先，必须要实现的方法：

    ```
    - (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section;
    ```

     这个方法用来设置tableView的每个分组的行数；

    ```
    - (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath;
    ```

    这个方法用来返回cell,就是用来控制每一行返回的内容；

    以上两个方法是必须实现的；

19. [NSString字符串比较](https://blog.csdn.net/xiaoyaoziqing/article/details/7165651)

20. ARC下，不显式指定任何属性关键字时，默认的关键字都有哪些

    - 对应基本数据类型默认关键字是：atomic,readwrite,assign

    - 对于普通的 Objective-C 对象：atomic,readwrite,strong

21. Oc 与 C++混合编译时文件后缀名改成什么

22. [iOS_使用ARC需要注意的问题](https://blog.csdn.net/xietao3/article/details/9716905)

23. [使用imagenamed方法创建uiimage对象,与普通的init方法有何区别](https://www.jianshu.com/p/585efd207869)

    imageNamed是会把读取到的image存在某个缓存里面，第二次读取相同图片的话系统就会直接从那个缓存中获取，从某种意义上好像一种优化，但是imageNamed读取到的那个图片似乎不会因为Memory Warning而释放，所以用这个会导致在内存不足的时候闪退。简单的说imageNamed采用了缓存机制，如果缓存中已加载了图片，直接从缓存读就行了，每次就不用再去读文件了，效率会更高 .

24. [self.name 和 _name的区别](https://www.jianshu.com/p/6053a490cdfb)

    1、self.name  是访问属性；_name是访问实例变量；

    2、在self.name=@"object"的时候，调用了setter方法 retainCount+1；_name=@"object"时，把object赋值给当前对象的name属性 retainCount+0；

    属性是实例变量加getter和setter方法的一个整合体，主要承担一个外部访问的一个接口。

    实例变量只能在本类中才可以访问，外部不能访问。

    使用原则：

    在类的内部访问变量的时候用下划线"_"

    其他类要访问这个类的变量时用"."

25. [在有了自动合成属性实例变量之后，@synthesize还有哪些使用场景](https://blog.csdn.net/dp948080952/article/details/52611348)

26. [webview加载html文件，如何调用原生态的代码的](https://blog.csdn.net/mengtianqq/article/details/70579277)

27. ios webview里加载了HTML页面，HTML 和APP原生功能如何相互调用 ，假设HTML未登录页面，点击密码输入框需要调用自定义乱序密码键盘，请简述：[webview HttpClient 怎么保持会话session统一](https://www.cnblogs.com/zylhxd/articles/10786843.html)

