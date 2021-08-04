# 计算机基础知识

OS.md、计网.md 资料底稿自于[CyC2018](https://github.com/CyC2018)/**[CS-Notes](https://github.com/CyC2018/CS-Notes)** 但删改幅度非常大，牛客IOS面经整理.md 是综合各方的整理

## [计网](https://github.com/Sophia-fez/SE-iOS-Job-Notes/blob/master/计网%20OS/计网.md)资料结构说明

- 一、概述，二、物理层，三、链路层，四、网络层；一到四章都是计网基础知识简单概括，面试基本不大会问到，不想翻书只想大概过一下的可以看一下，时间来不及完全不看也没关系
- 五、传输层，面试会问到TCP、UDP区别，三次握手四次握手，TCP如何保证可靠通信（滑动窗口、流量控制、拥塞控制），但不会问的很深入
- **六、应用层，这章是面试问的重点部分**，毕竟客户端开发最重要的其实就是交互和通信，所以一般大概会有这样的循序渐进过程：**web页面请求过程、dns原理、get和post、https原理、https再深入拓展的CA证书和加密协议**，亲测工作日常就是要和证书、https通信打交道的，所以是面试的重点也无可厚非

## [OS](https://github.com/Sophia-fez/SE-iOS-Job-Notes/blob/master/计网%20OS/OS.md)资料结构说明

- 一、概述，五、设备管理，六、链接，这几章相对不重要
- 二、进程管理，重点，一定会问进程线程的区别，进程通信，进程同步
- 三、死锁，重点但内容很少，一定会问死锁的必要条件和处理方式
- 四、内存管理，虚拟内存、分页、分段、段页式，这章内容比较多大概了解一下能掰扯两句就行，有时候还是会问的

## [牛客IOS面经整理](https://github.com/Sophia-fez/SE-iOS-Job-Notes/blob/master/面试相关/牛客IOS面经整理.md)

[OS](https://github.com/Sophia-fez/SE-iOS-Job-Notes/blob/master/计网%20OS/OS.md)和[计网](https://github.com/Sophia-fez/SE-iOS-Job-Notes/blob/master/计网%20OS/计网.md)是这两门计算机基础课程的系统性资料，里面也对面试重点部分进行了标注，而[牛客IOS面经整理](https://github.com/Sophia-fez/SE-iOS-Job-Notes/blob/master/面试相关/牛客IOS面经整理.md)则是将其中的重点挑了出来形成的文档，不仅包含OS和计网的重点部分，还包含了iOS相关基础概念的一个重要模块，还包含了部分数据结构、编程语言、智力题的稍许资料。

## 增删记录

### 目前的主要修改：

1. 格式、排版
2. 计网：
   - 码分复用，增加例子
   - RIP距离向量算法，增加RIP更新路由表的详细过程
   - 增加 DNS 整块内容，详细叙述了DNS解析过程（面试重点考察内容）
   - 增加HTTP、HTTPS整块内容（面试重点考察内容）

### OS待深入研究的知识点：

因为不是面试重点考察（没见过要手撕这些代码的），暂不予补充，自行百度也很简单或者写

1. 进程同步（代码已有，找一下以前自己写的还有书上的）
   - 生产者与消费者
   - 哲学家进餐
   - 读写者问题
2. 页面置换算法（代码找一下）
   - OPT最佳
   - LRU最近最久未使用
   - NRU最近未使用
   - FIFP先进先出
   - 第二次机会算法
   - 时钟Clock
4. 磁盘调度算法（代码找一下）
   - FCFS先来先服务
   - SSTF最短寻找优先
   - SCAN电梯算法

