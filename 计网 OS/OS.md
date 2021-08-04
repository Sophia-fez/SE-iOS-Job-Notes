[TOC]

# OS：一、概述

## 1. 基本特征

1. **并发和并行**

    并发：宏观上在一段时间内能同时运行多个程序。OS引入进程和线程，使程序能够并发运行。

    并行：同一时刻能运行多个指令。并行需要硬件支持，如多流水线、多核处理器或者分布式计算系统。

2. **共享：**系统中的资源可以被多个并发进程共同使用。

    共享的两种方式：互斥共享、同时共享。

    互斥共享的资源称为临界资源，例如打印机等，在同一时刻只允许一个进程访问，需要用同步机制来实现互斥访问。

3. **虚拟：**虚拟技术把一个物理实体转换为多个逻辑实体。

  两种虚拟技术：

  - 时分复用：多个进程能在同一个处理器上并发执行，让每个进程轮流占用处理器，每次只执行一小个时间片并快速切换。
  - 空分复用：虚拟内存，将物理内存抽象为地址空间，每个进程都有各自的地址空间。地址空间的页被映射到物理内存，地址空间的页并不需要全部在物理内存中，当使用到一个没有在物理内存的页时，执行页面置换算法将该页置换到内存中。

4. **异步：**进程不是一次性执行完毕，而是走走停停，以不可知的速度向前推进。

## 2. 基本功能

1. 进程管理：进程控制、进程同步、进程通信、死锁处理、处理机调度
2. 内存管理：内存分配、地址映射、内存保护与共享、虚拟内存
3. 文件管理：文件存储空间管理、目录管理、文件读写管理和保护
4. 设备管理：缓冲管理、设备分配、设备处理、虛拟设备；完成用户的 I/O 请求，方便用户使用各种设备，并提高设备的利用率。

## 3. 系统调用

如果一个进程在用户态需要使用内核态的功能，就进行系统调用从而陷入内核，由操作系统代为完成。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/tGPV0.png" width="600"/> </div><br>

## 4. 大内核和微内核

1. **大内核**

	将操作系统功能作为一个紧密结合的整体放到内核。由于各模块共享信息，因此有很高的性能。

2. **微内核**
   
    - 由于操作系统不断复杂，因此将一部分操作系统功能移出内核，从而降低内核的复杂性。移出的部分根据分层的原则划分成若干服务，相互独立。
    - 在微内核结构下，操作系统被划分成小的、定义良好的模块，只有微内核这一个模块运行在内核态，其余模块运行在用户态。
    - 因为需要频繁地在用户态和核心态之间进行切换，所以会有一定的性能损失。
	
	<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/2_14_microkernelArchitecture.jpg"/> </div><br>

## 5. 中断分类

1. **外中断**

	由 **CPU 执行指令以外**的事件引起，如 I/O 完成中断，表示设备输入/输出处理已经完成，处理器能够发送下一个输入/输出请求。此外还有时钟中断、控制台中断等。

2. **异常：**由 **CPU 执行指令的内部**事件引起，如非法操作码、地址越界、算术溢出等。
3. **陷入：**在用户程序中使用系统调用。

# 二、进程管理

## 1. 进程与线程的区别（重点）

1. **进程：资源分配的基本单位**

    进程控制块 (Process Control Block, PCB) 描述进程的基本信息和运行状态，所谓的创建进程和撤销进程，都是指对 PCB 的操作。下图显示了 4 个程序创建了 4 个进程，这 4 个进程可以并发地执行。

    <div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/a6ac2b08-3861-4e85-baa8-382287bfee9f.png"/> </div><br>

2. **线程：独立调度的基本单位**

    **一个进程中可以有多个线程，它们共享进程资源。**

    QQ 和浏览器是两个进程，浏览器进程里面有很多线程，例如 HTTP 请求线程、事件响应线程、渲染线程等等，线程的并发执行使得在浏览器中点击一个新链接从而发起 HTTP 请求时，浏览器还可以响应用户的其它事件。

    <div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/3cd630ea-017c-488d-ad1d-732b4efeddf5.png"/> </div><br>

3. **区别**（挑着说）

    - 拥有资源
    进程是资源分配的基本单位，但是线程不拥有资源，线程可以访问隶属进程的资源。
    - 调度
    线程是独立调度的基本单位，在同一进程中，线程的切换不会引起进程切换，从一个进程中的线程切换到另一个进程中的线程时，会引起进程切换。
    - 通信方面
    线程间可以通过直接读写同一进程中的数据进行通信，但是进程通信需要借助 IPC（Inter-Process Communication，进程间通信）。
    - 系统开销
    由于创建或撤销进程时，系统都要为之分配或回收资源，如内存空间、I/O 设备等，所付出的开销远大于创建或撤销线程时的开销。类似地，在进行进程切换时，涉及当前执行进程 CPU 环境的保存及新调度进程 CPU 环境的设置，而线程切换时只需保存和设置少量寄存器内容，开销很小。

## 2. 进程状态的切换

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/ProcessState.png" width="500"/> </div><br>

- 就绪状态（ready）：等待被调度
- 运行状态（running）
- 阻塞状态（waiting）：等待资源

应该注意以下内容：

- 只有就绪态和运行态可以相互转换，其它的都是单向转换。就绪状态的进程通过调度算法从而获得 CPU 时间，转为运行状态；而运行状态的进程，在分配给它的 CPU 时间片用完之后就会转为就绪状态，等待下一次调度。
- 阻塞状态是缺少需要的资源从而由运行状态转换而来，但是该资源不包括 CPU 时间，缺少 CPU 时间会从运行态转换为就绪态。

## 3. 进程调度算法（会问）

不同环境的调度算法目标不同，因此需要针对不同环境来讨论调度算法。

### 3.1. 批处理系统

批处理系统没有太多的用户操作，在该系统中，调度算法目标是保证吞吐量和周转时间（从提交到终止的时间）。

1. **先来先服务 first-come first-serverd（FCFS）**

    非抢占式的调度算法，按照请求的顺序进行调度。

    有利于长作业，但不利于短作业，因为短作业必须一直等待前面的长作业执行完毕才能执行，而长作业又需要执行很长时间，造成了短作业等待时间过长。

2. **短作业优先 shortest job first（SJF）**

    非抢占式的调度算法，按估计运行时间最短的顺序进行调度。

    长作业有可能会饿死，处于一直等待短作业执行完毕的状态。因为如果一直有短作业到来，那么长作业永远得不到调度。

3. **最短剩余时间优先 shortest remaining time next（SRTN）**

    最短作业优先的抢占式版本，按剩余运行时间的顺序进行调度。 当一个新的作业到达时，其整个运行时间与当前进程的剩余时间作比较。如果新的进程需要的时间更少，则挂起当前进程，运行新的进程。否则新的进程等待。

### 3.2. 交互式系统

交互式系统有大量的用户交互操作，在该系统中调度算法的目标是快速地进行响应。

1. **时间片轮转**

    将所有就绪进程按 FCFS 的原则排成一个队列，每次调度时，把 CPU 时间分配给队首进程，该进程可以执行一个时间片。当时间片用完时，由计时器发出时钟中断，调度程序便停止该进程的执行，并将它送往就绪队列的末尾，同时继续把 CPU 时间分配给队首的进程。

    时间片轮转算法的效率和时间片的大小有很大关系：

    - 因为进程切换都要保存进程的信息并且载入新进程的信息，如果时间片太小，会导致进程切换得太频繁，在进程切换上就会花过多时间。
    - 而如果时间片过长，那么实时性就不能得到保证。

    <div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/8c662999-c16c-481c-9f40-1fdba5bc9167.png"/> </div><br>

2. **优先级调度**

    为每个进程分配一个优先级，按优先级进行调度。

    为了防止低优先级的进程永远等不到调度，可以随着时间的推移增加等待进程的优先级。

3. **多级反馈队列**

    一个进程需要执行 100 个时间片，如果采用时间片轮转调度算法，那么需要交换 100 次。

    多级队列是为这种需要连续执行多个时间片的进程考虑，它设置了多个队列，每个队列时间片大小都不同，例如 1,2,4,8,..。进程在第一个队列没执行完，就会被移到下一个队列。这种方式下，之前的进程只需要交换 7 次。

    每个队列优先权也不同，最上面的优先权最高。因此只有上一个队列没有进程在排队，才能调度当前队列上的进程。

    可以将这种调度算法看成是时间片轮转调度算法和优先级调度算法的结合。

    <div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/042cf928-3c8e-4815-ae9c-f2780202c68f.png"/> </div><br>

### 3.3 实时系统

实时系统要求一个请求在一个确定时间内得到响应。

分为硬实时和软实时，前者必须满足绝对的截止时间，后者可以容忍一定的超时。

## 4. CPU资源调度（进程调度+虚拟内存页面置换）

**时间片分配（进程调度）：先来先服务FCFS、短作业优先SJF进程调度、时间片轮转RR**

**内存分配管理（段式、页式，段页式），虚拟内存页面置换算法：先进先出FIFO、最佳OPI、最近最久未使用LRU、最近未使用NRU**

## 5. 进程通信（管道、FIFO、消息队列、信号量、共同存储、套接字Socket）（重点）

进程同步与进程通信很容易混淆，它们的区别在于：

- **进程同步：控制多个进程按一定顺序执行；**
- **进程通信：进程间传输信息。**

进程通信是一种手段，而进程同步是一种目的。也可以说，为了能够达到进程同步的目的，需要让进程进行通信，传输一些进程同步所需要的信息。

1. **管道**

   管道是通过调用 pipe 函数创建的，fd[0] 用于读，fd[1] 用于写。

```c
#include <unistd.h>
int pipe(int fd[2]);
```

它具有以下限制：

- 只支持半双工通信（单向交替传输）；
- 只能在父子进程或者兄弟进程中使用。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/53cd9ade-b0a6-4399-b4de-7f1fbd06cdfb.png"/> </div><br>

2. **FIFO**

   也称为命名管道，去除了管道只能在父子进程中使用的限制。

```c
#include <sys/stat.h>
int mkfifo(const char *path, mode_t mode);
int mkfifoat(int fd, const char *path, mode_t mode);
```

FIFO 常用于客户-服务器应用程序中，FIFO 用作汇聚点，在客户进程和服务器进程之间传递数据。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/2ac50b81-d92a-4401-b9ec-f2113ecc3076.png"/> </div><br>

3. **消息队列**

   相比于 FIFO，消息队列具有以下优点：

    - 消息队列可以独立于读写进程存在，从而避免了 FIFO 中同步管道的打开和关闭时可能产生的困难；
    - 避免了 FIFO 的同步阻塞问题，不需要进程自己提供同步方法；
    - 读进程可以根据消息类型有选择地接收消息，而不像 FIFO 那样只能默认地接收。

4. **信号量**

   它是一个计数器，用于为多个进程提供对共享数据对象的访问。

5. **共享存储**

   允许多个进程共享一个给定的存储区。因为数据不需要在进程之间复制，所以这是最快的一种 IPC。
   需要使用信号量用来同步对共享存储的访问。

   多个进程可以将同一个文件映射到它们的地址空间从而实现共享内存。另外 XSI 共享内存不是使用文件，而是使用内存的匿名段。

6. **套接字**

   与其它通信机制不同的是，它可用于不同机器间的进程通信。

## 6. 进程同步（临界区、同步与互斥、信号量、管程）（重点）

1. **临界区**

    对临界资源进行访问的那段代码称为临界区。

    为了互斥访问临界资源，每个进程在进入临界区之前，需要先进行检查。
    
```html
// entry section
// critical section;
// exit section
```

2. **同步与互斥**

    - 同步：多个进程因为合作产生的直接制约关系，使得进程有一定的先后执行关系。
    - 互斥：多个进程在同一时刻只有一个进程能进入临界区。

3. **信号量**

    信号量（Semaphore）是一个整型变量，可以对其执行 down 和 up 操作，也就是常见的 P 和 V 操作。

    down 和 up 操作需要被设计成原语，不可分割，通常的做法是在执行这些操作的时候屏蔽中断。
    如果信号量的取值只能为 0 或者 1，那么就成为了   **互斥量（Mutex）**  ，0 表示临界区已经加锁，1 表示临界区解锁。

    -   **down**   : 如果信号量大于 0 ，执行 -1 操作；如果信号量等于 0，进程睡眠，等待信号量大于 0；
    -   **up**  ：对信号量执行 +1 操作，唤醒睡眠的进程让其完成 down 操作。


```c
typedef int semaphore;
semaphore mutex = 1;
void P1() {
    down(&mutex);
    // 临界区
    up(&mutex);
}

void P2() {
    down(&mutex);
    // 临界区
    up(&mutex);
}
```

**例：使用信号量实现生产者-消费者问题**

问题描述：使用一个缓冲区来保存物品，只有缓冲区没有满，生产者才可以放入物品；只有缓冲区不为空，消费者才可以拿走物品。

因为缓冲区属于临界资源，因此需要使用一个互斥量 mutex 来控制对缓冲区的互斥访问。

为了同步生产者和消费者的行为，需要记录缓冲区中物品的数量。数量可以使用信号量来进行统计，这里需要使用两个信号量：empty 记录空缓冲区的数量，full 记录满缓冲区的数量。其中，empty 信号量是在生产者进程中使用，当 empty 不为 0 时，生产者才可以放入物品；full 信号量是在消费者进程中使用，当 full 信号量不为 0 时，消费者才可以取走物品。

注意，不能先对缓冲区进行加锁，再测试信号量。也就是说，不能先执行 down(mutex) 再执行 down(empty)。如果这么做了，那么可能会出现这种情况：生产者对缓冲区加锁后，执行 down(empty) 操作，发现 empty = 0，此时生产者睡眠。消费者不能进入临界区，因为生产者对缓冲区加锁了，消费者就无法执行 up(empty) 操作，empty 永远都为 0，导致生产者永远等待下，不会释放锁，消费者因此也会永远等待下去。

```c
#define N 100
typedef int semaphore;
semaphore mutex = 1;
semaphore empty = N;
semaphore full = 0;

void producer() {
    while(TRUE) {
        int item = produce_item();
        down(&empty);
        down(&mutex);
        insert_item(item);
        up(&mutex);
        up(&full);
    }
}

void consumer() {
    while(TRUE) {
        down(&full);
        down(&mutex);
        int item = remove_item();
        consume_item(item);
        up(&mutex);
        up(&empty);
    }
}
```

4. **管程**

    使用信号量机制实现的生产者消费者问题需要客户端代码做很多控制，而管程把控制的代码独立出来，不仅不容易出错，也使得客户端代码调用更容易。

    c 语言不支持管程，下面的示例代码使用了类 Pascal 语言来描述管程。示例代码的管程提供了 insert() 和 remove() 方法，客户端代码通过调用这两个方法来解决生产者-消费者问题。

```pascal
monitor ProducerConsumer
    integer i;
    condition c;

    procedure insert();
    begin
        // ...
    end;

    procedure remove();
    begin
        // ...
    end;
end monitor;
```

管程有一个重要特性：在一个时刻只能有一个进程使用管程。进程在无法继续执行的时候不能一直占用管程，否则其它进程永远不能使用管程。

管程引入了   **条件变量**   以及相关的操作：**wait()** 和 **signal()** 来实现同步操作。对条件变量执行 wait() 操作会导致调用进程阻塞，把管程让出来给另一个进程持有。signal() 操作用于唤醒被阻塞的进程。

<font size=3>  **使用管程实现生产者-消费者问题**  </font><br>

```pascal
// 管程
monitor ProducerConsumer
    condition full, empty;
    integer count := 0;
    condition c;

    procedure insert(item: integer);
    begin
        if count = N then wait(full);
        insert_item(item);
        count := count + 1;
        if count = 1 then signal(empty);
    end;

    function remove: integer;
    begin
        if count = 0 then wait(empty);
        remove = remove_item;
        count := count - 1;
        if count = N -1 then signal(full);
    end;
end monitor;

// 生产者客户端
procedure producer
begin
    while true do
    begin
        item = produce_item;
        ProducerConsumer.insert(item);
    end
end;

// 消费者客户端
procedure consumer
begin
    while true do
    begin
        item = ProducerConsumer.remove;
        consume_item(item);
    end
end;
```

## 7.线程安全（这部分比较迷）

线程安全性问题出现的三个必要条件：

- 多线程环境下

- 多个线程共享同一个资源

- 对资源进行非原子性操作

解决线程安全的四种方式：

- synchronized锁（偏向锁，轻量级锁，重量级锁）

- volatile英[ˈvɒlətaɪl]锁，只能保证线程之间的可见性，但不能保证数据的原子性

- jdk1.5并发包中提供的Atomic原子类

- Lock锁

- 多实例、或者是多副本（ThreadLocal）：对应着思路1 2，ThreadLocal可以为每个线程的维护一个私有的本地变量，可参考[java线程副本–ThreadLocal](http://blog.csdn.net/jinggod/article/details/78268020)；

一般说来，确保线程安全的方法有这几个：**竞争与原子操作、同步与锁、可重入、过度优化**。

**竞争与原子操作**
多个线程同时访问和修改一个数据，可能造成很严重的后果。出现严重后果的原因是很多操作被操作系统编译为汇编代码之后不止一条指令，因此在执行的时候可能执行了一半就被调度系统打断了而去执行别的代码了。一般将单指令的操作称为原子的(Atomic)，因为不管怎样，单条指令的执行是不会被打断的。

因此，为了避免出现多线程操作数据的出现异常，Linux系统提供了一些常用操作的原子指令，确保了线程的安全。但是，它们只适用于比较简单的场合，在复杂的情况下就要选用其他的方法了。

**同步与锁**
为了避免多个线程同时读写一个数据而产生不可预料的后果，开发人员要将各个线程对同一个数据的访问同步，也就是说，在一个线程访问数据未结束的时候，其他线程不得对同一个数据进行访问。

同步的最常用的方法是使用锁(Lock)，它是一种非强制机制，每个线程在访问数据或资源之前首先试图获取锁，并在访问结束之后释放锁；在锁已经被占用的时候试图获取锁时，线程会等待，直到锁重新可用。

二元信号量是最简单的一种锁，它只有两种状态：占用与非占用，它适合只能被唯一一个线程独占访问的资源。对于允许多个线程并发访问的资源，要使用多元信号量(简称信号量)。

**可重入**
一个函数被重入，表示这个函数没有执行完成，但由于外部因素或内部因素，又一次进入该函数执行。一个函数称为可重入的，表明该函数被重入之后不会产生任何不良后果。可重入是并发安全的强力保障，一个可重入的函数可以在多线程环境下放心使用。

**过度优化**
在很多情况下，即使我们合理地使用了锁，也不一定能够保证线程安全，因此，我们可能对代码进行过度的优化以确保线程安全。

我们可以使用volatile关键字试图阻止过度优化，它可以做两件事：第一，阻止编译器为了提高速度将一个变量缓存到寄存器而不写回；第二，阻止编译器调整操作volatile变量的指令顺序。

在另一种情况下，CPU的乱序执行让多线程安全保障的努力变得很困难，通常的解决办法是调用CPU提供的一条常被称作barrier的指令，它会阻止CPU将该指令之前的指令交换到barrier之后，反之亦然。

**锁，知道的锁和用法**，程序锁，乐观锁，悲观锁，单线程锁，共享锁，非共享锁还有什么锁

## 8. 经典同步问题（可以不看）

生产者和消费者问题前面已经讨论过了。

### 8.1 哲学家进餐问题

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/a9077f06-7584-4f2b-8c20-3a8e46928820.jpg"/> </div><br>

五个哲学家围着一张圆桌，每个哲学家面前放着食物。哲学家的生活有两种交替活动：吃饭以及思考。当一个哲学家吃饭时，需要先拿起自己左右两边的两根筷子，并且一次只能拿起一根筷子。

下面是一种错误的解法，如果所有哲学家同时拿起左手边的筷子，那么所有哲学家都在等待其它哲学家吃完并释放自己手中的筷子，导致死锁。

```c
#define N 5

void philosopher(int i) {
    while(TRUE) {
        think();
        take(i);       // 拿起左边的筷子
        take((i+1)%N); // 拿起右边的筷子
        eat();
        put(i);
        put((i+1)%N);
    }
}
```

为了防止死锁的发生，可以设置两个条件：

- 必须同时拿起左右两根筷子；
- 只有在两个邻居都没有进餐的情况下才允许进餐。

```c
#define N 5
#define LEFT (i + N - 1) % N // 左邻居
#define RIGHT (i + 1) % N    // 右邻居
#define THINKING 0
#define HUNGRY   1
#define EATING   2
typedef int semaphore;
int state[N];                // 跟踪每个哲学家的状态
semaphore mutex = 1;         // 临界区的互斥，临界区是 state 数组，对其修改需要互斥
semaphore s[N];              // 每个哲学家一个信号量

void philosopher(int i) {
    while(TRUE) {
        think(i);
        take_two(i);
        eat(i);
        put_two(i);
    }
}

void take_two(int i) {
    down(&mutex);
    state[i] = HUNGRY;
    check(i);
    up(&mutex);
    down(&s[i]); // 只有收到通知之后才可以开始吃，否则会一直等下去
}

void put_two(i) {
    down(&mutex);
    state[i] = THINKING;
    check(LEFT); // 尝试通知左右邻居，自己吃完了，你们可以开始吃了
    check(RIGHT);
    up(&mutex);
}

void eat(int i) {
    down(&mutex);
    state[i] = EATING;
    up(&mutex);
}

// 检查两个邻居是否都没有用餐，如果是的话，就 up(&s[i])，使得 down(&s[i]) 能够得到通知并继续执行
void check(i) {         
    if(state[i] == HUNGRY && state[LEFT] != EATING && state[RIGHT] !=EATING) {
        state[i] = EATING;
        up(&s[i]);
    }
}
```

### 8.2 读者-写者问题

允许多个进程同时对数据进行读操作，但是不允许读和写以及写和写操作同时发生。

一个整型变量 count 记录在对数据进行读操作的进程数量，一个互斥量 count_mutex 用于对 count 加锁，一个互斥量 data_mutex 用于对读写的数据加锁。

```c
typedef int semaphore;
semaphore count_mutex = 1;
semaphore data_mutex = 1;
int count = 0;

void reader() {
    while(TRUE) {
        down(&count_mutex);
        count++;
        if(count == 1) down(&data_mutex); // 第一个读者需要对数据进行加锁，防止写进程访问
        up(&count_mutex);
        read();
        down(&count_mutex);
        count--;
        if(count == 0) up(&data_mutex);
        up(&count_mutex);
    }
}

void writer() {
    while(TRUE) {
        down(&data_mutex);
        write();
        up(&data_mutex);
    }
}
```

以下内容由 [@Bandi Yugandhar](https://github.com/yugandharbandi) 提供。

The first case may result Writer to starve. This case favous Writers i.e no writer, once added to the queue, shall be kept waiting longer than absolutely necessary(only when there are readers that entered the queue before the writer).

```c
int readcount, writecount;                   //(initial value = 0)
semaphore rmutex, wmutex, readLock, resource; //(initial value = 1)

//READER
void reader() {
<ENTRY Section>
 down(&readLock);                 //  reader is trying to enter
 down(&rmutex);                  //   lock to increase readcount
  readcount++;                 
  if (readcount == 1)          
   down(&resource);              //if you are the first reader then lock  the resource
 up(&rmutex);                  //release  for other readers
 up(&readLock);                 //Done with trying to access the resource

<CRITICAL Section>
//reading is performed

<EXIT Section>
 down(&rmutex);                  //reserve exit section - avoids race condition with readers
 readcount--;                       //indicate you're leaving
  if (readcount == 0)          //checks if you are last reader leaving
   up(&resource);              //if last, you must release the locked resource
 up(&rmutex);                  //release exit section for other readers
}

//WRITER
void writer() {
  <ENTRY Section>
  down(&wmutex);                  //reserve entry section for writers - avoids race conditions
  writecount++;                //report yourself as a writer entering
  if (writecount == 1)         //checks if you're first writer
   down(&readLock);               //if you're first, then you must lock the readers out. Prevent them from trying to enter CS
  up(&wmutex);                  //release entry section

<CRITICAL Section>
 down(&resource);                //reserve the resource for yourself - prevents other writers from simultaneously editing the shared resource
  //writing is performed
 up(&resource);                //release file

<EXIT Section>
  down(&wmutex);                  //reserve exit section
  writecount--;                //indicate you're leaving
  if (writecount == 0)         //checks if you're the last writer
   up(&readLock);               //if you're last writer, you must unlock the readers. Allows them to try enter CS for reading
  up(&wmutex);                  //release exit section
}
```

We can observe that every reader is forced to acquire ReadLock. On the otherhand, writers doesn’t need to lock individually. Once the first writer locks the ReadLock, it will be released only when there is no writer left in the queue.

From the both cases we observed that either reader or writer has to starve. Below solutionadds the constraint that no thread shall be allowed to starve; that is, the operation of obtaining a lock on the shared data will always terminate in a bounded amount of time.

```source-c
int readCount;                  // init to 0; number of readers currently accessing resource

// all semaphores initialised to 1
Semaphore resourceAccess;       // controls access (read/write) to the resource
Semaphore readCountAccess;      // for syncing changes to shared variable readCount
Semaphore serviceQueue;         // FAIRNESS: preserves ordering of requests (signaling must be FIFO)

void writer()
{ 
    down(&serviceQueue);           // wait in line to be servicexs
    // <ENTER>
    down(&resourceAccess);         // request exclusive access to resource
    // </ENTER>
    up(&serviceQueue);           // let next in line be serviced

    // <WRITE>
    writeResource();            // writing is performed
    // </WRITE>

    // <EXIT>
    up(&resourceAccess);         // release resource access for next reader/writer
    // </EXIT>
}

void reader()
{ 
    down(&serviceQueue);           // wait in line to be serviced
    down(&readCountAccess);        // request exclusive access to readCount
    // <ENTER>
    if (readCount == 0)         // if there are no readers already reading:
        down(&resourceAccess);     // request resource access for readers (writers blocked)
    readCount++;                // update count of active readers
    // </ENTER>
    up(&serviceQueue);           // let next in line be serviced
    up(&readCountAccess);        // release access to readCount

    // <READ>
    readResource();             // reading is performed
    // </READ>

    down(&readCountAccess);        // request exclusive access to readCount
    // <EXIT>
    readCount--;                // update count of active readers
    if (readCount == 0)         // if there are no readers left:
        up(&resourceAccess);     // release resource access for all
    // </EXIT>
    up(&readCountAccess);        // release access to readCount
}

```

# 三、死锁（重点）

## 1. 死锁的必要条件

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/c037c901-7eae-4e31-a1e4-9d41329e5c3e.png"/> </div><br>

- **互斥：**每个资源要么已经分配给了一个进程，要么就是可用的。
- **占有和等待**：已经得到了某个资源的进程可以再请求新的资源。
- **不可抢占：**已经分配给一个进程的资源不能强制性地被抢占，它只能被占有它的进程显式地释放。
- **环路等待：**有两个或者两个以上的进程组成一条环路，该环路中的每个进程都在等待下一个进程所占有的资源。

## 2. 死锁处理方法

- 鸵鸟策略
- 死锁检测与死锁恢复
- 死锁预防
- 死锁避免

### 鸵鸟策略

把头埋在沙子里，假装根本没发生问题。

因为解决死锁问题的代价很高，因此鸵鸟策略这种不采取任务措施的方案会获得更高的性能。

当发生死锁时不会对用户造成多大影响，或发生死锁的概率很低，可以采用鸵鸟策略。

大多数操作系统，包括 Unix，Linux 和 Windows，处理死锁问题的办法仅仅是忽略它。

### 死锁检测与死锁恢复

不试图阻止死锁，而是当检测到死锁发生时，采取措施进行恢复。

1. **每种类型一个资源的死锁检测**

    <div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/b1fa0453-a4b0-4eae-a352-48acca8fff74.png"/> </div><br>

    上图为资源分配图，其中方框表示资源，圆圈表示进程。资源指向进程表示该资源已经分配给该进程，进程指向资源表示进程请求获取该资源。

    图 a 可以抽取出环，如图 b，它满足了环路等待条件，因此会发生死锁。

    每种类型一个资源的死锁检测算法是通过检测有向图是否存在环来实现，从一个节点出发进行深度优先搜索，对访问过的节点进行标记，如果访问了已经标记的节点，就表示有向图存在环，也就是检测到死锁的发生。

2. **每种类型多个资源的死锁检测**

    <div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/e1eda3d5-5ec8-4708-8e25-1a04c5e11f48.png"/> </div><br>

    上图中，有三个进程四个资源，每个数据代表的含义如下：

    - E 向量：资源总量
    - A 向量：资源剩余量
    - C 矩阵：每个进程所拥有的资源数量，每一行都代表一个进程拥有资源的数量
    - R 矩阵：每个进程请求的资源数量

    进程 P<sub>1</sub> 和 P<sub>2</sub> 所请求的资源都得不到满足，只有进程 P<sub>3</sub> 可以，让 P<sub>3</sub> 执行，之后释放 P<sub>3</sub> 拥有的资源，此时 A = (2 2 2 0)。P<sub>2</sub> 可以执行，执行后释放 P<sub>2</sub> 拥有的资源，A = (4 2 2 1) 。P<sub>1</sub> 也可以执行。所有进程都可以顺利执行，没有死锁。

    算法总结如下：

    每个进程最开始时都不被标记，执行过程有可能被标记。当算法结束时，任何没有被标记的进程都是死锁进程。

    - 寻找一个没有标记的进程 P<sub>i</sub>，它所请求的资源小于等于 A。
    - 如果找到了这样一个进程，那么将 C 矩阵的第 i 行向量加到 A 中，标记该进程，并转回 1。
    - 如果没有这样一个进程，算法终止。


3. **死锁恢复**

    - 利用抢占恢复
    - 利用回滚恢复
    - 通过杀死进程恢复

### 死锁预防

在程序运行之前预防发生死锁。

- **破坏互斥条件**，例如假脱机打印机技术允许若干个进程同时输出，唯一真正请求物理打印机的进程是打印机守护进程。
- **破坏占有和等待条件**，一种实现方式是规定所有进程在开始执行前请求所需要的全部资源。
- **破坏不可抢占条件**
- **破坏环路等待**，给资源统一编号，进程只能按编号顺序来请求资源。

### 死锁避免

在程序运行时避免发生死锁。

1. **安全状态**

    <div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/ed523051-608f-4c3f-b343-383e2d194470.png"/> </div><br>

    图 a 的第二列 Has 表示已拥有的资源数，第三列 Max 表示总共需要的资源数，Free 表示还有可以使用的资源数。从图 a 开始出发，先让 B 拥有所需的所有资源（图 b），运行结束后释放 B，此时 Free 变为 5（图 c）；接着以同样的方式运行 C 和 A，使得所有进程都能成功运行，因此可以称图 a 所示的状态时安全的。

    定义：如果没有死锁发生，并且即使所有进程突然请求对资源的最大需求，也仍然存在某种调度次序能够使得每一个进程运行完毕，则称该状态是安全的。

    安全状态的检测与死锁的检测类似，因为安全状态必须要求不能发生死锁。下面的银行家算法与死锁检测算法非常类似，可以结合着做参考对比。

2. **单个资源的银行家算法**

    一个小城镇的银行家，他向一群客户分别承诺了一定的贷款额度，算法要做的是判断对请求的满足是否会进入不安全状态，如果是，就拒绝请求；否则予以分配。

    <div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/d160ec2e-cfe2-4640-bda7-62f53e58b8c0.png"/> </div><br>

    上图 c 为不安全状态，因此算法会拒绝之前的请求，从而避免进入图 c 中的状态。

3. **多个资源的银行家算法**

    <div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/62e0dd4f-44c3-43ee-bb6e-fedb9e068519.png"/> </div><br>

    上图中有五个进程，四个资源。左边的图表示已经分配的资源，右边的图表示还需要分配的资源。最右边的 E、P 以及 A 分别表示：总资源、已分配资源以及可用资源，注意这三个为向量，而不是具体数值，例如 A=(1020)，表示 4 个资源分别还剩下 1/0/2/0。

    检查一个状态是否安全的算法如下：

    - 查找右边的矩阵是否存在一行小于等于向量 A。如果不存在这样的行，那么系统将会发生死锁，状态是不安全的。
    - 假若找到这样一行，将该进程标记为终止，并将其已分配资源加到 A 中。
    - 重复以上两步，直到所有进程都标记为终止，则状态时安全的。

    如果一个状态不是安全的，需要拒绝进入这个状态。

# 四、内存管理（了解一下）

## 1. 虚拟内存

虚拟内存的目的是为了让物理内存扩充成更大的逻辑内存，从而让程序获得更多的可用内存。

为了更好的管理内存，操作系统将内存抽象成地址空间。**每个程序拥有自己的地址空间，这个地址空间被分割成多个块（每块相等），每一块称为一页。这些页被映射到物理内存，但不需要映射到连续的物理内存，也不需要所有页都必须在物理内存中。**当程序引用到不在物理内存中的页时**（缺页中断）**，由硬件执行必要的映射，将缺失的部分装入物理内存并重新执行失败的指令。

虚拟内存允许程序不用将地址空间中的每一页都映射到物理内存，即说一个程序不需要全部调入内存就可以运行，这使得有限的内存运行大程序成为可能。例如有一台计算机可以产生 16 位地址，那么一个程序的地址空间范围是 0\~64K。该计算机只有 32KB 的物理内存，虚拟内存技术允许该计算机运行一个 64K 大小的程序。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/7b281b1e-0595-402b-ae35-8c91084c33c1.png"/> </div><br>

## 2. 分页

进程中的块称为页，内存中的块称为页框，外存中的块就叫块。

一个虚拟地址分成两个部分，一部分存储**页号**，一部分**页内偏移量**

<img src="../image\QQ截图20201026165120.png" alt="QQ截图20201026165120" style="zoom: 75%;" />

页表：系统为每个进程建立一张页表，页表由页表项组成，每一个页表项分为**页号**和**物理内存中的块号**两部分，物理内存中的块号与页内偏移量共同组成物理地址

<img src="../image\QQ截图20201026165402.png" alt="QQ截图20201026165402" style="zoom:75%;" />

<img src="../image\QQ截图20201026165425.png" alt="QQ截图20201026165425" style="zoom:75%;" />

<img src="../image\QQ截图20201026165459.png" alt="QQ截图20201026165459" style="zoom:75%;" />

分页管理系统中地址空间是一维的，页表不能太大，否则内存利用率会降低

## 3. 分段

分段的做法是把每个表分成段，一个段构成一个独立的地址空间。**每个段的长度可以不同，并且可以动态增长。**段内地址空间要求连续，段间不要求连续。

<img src="../image\QQ截图20201026165530.png" alt="QQ截图20201026165530" style="zoom:75%;" />

<img src="../image\QQ截图20201026165737.png" alt="QQ截图20201026165737" style="zoom:75%;" />

<img src="../image\QQ截图20201026165741.png" alt="QQ截图20201026165741" style="zoom:75%;" />

<img src="../image\QQ截图20201026165639.png" alt="QQ截图20201026165639" style="zoom:75%;" />

## 4. 段页式

**程序的地址空间划分成多个拥有独立地址空间的段，每个段上的地址空间划分成大小相同的页。**这样既拥有分段系统的共享和保护，又拥有分页系统的虚拟内存功能。系统为每个进程建立一张段表，每个分段都有一张页表。在一个进程中，段表只有一个，但页表可能有多个。

<img src="../image\QQ截图20201026165827.png" alt="QQ截图20201026165827" style="zoom:75%;" />

<img src="../image\QQ截图20201026165822.png" alt="QQ截图20201026165822" style="zoom:75%;" />

<img src="../image\QQ截图20201026165923.png" alt="QQ截图20201026165923" style="zoom:75%;" />

## 5. 分页与分段的比较

- 对程序员的透明性：分页透明，分段需要程序员显式划分每个段。
- 地址空间的维度：分页是一维地址空间，分段是二维的。

- 大小是否可以改变：**页的大小不可变，段的大小可以动态改变。**

- 出现的原因：**分页主要用于实现虚拟内存，从而获得更大的地址空间；分段主要是为了使程序和数据可以被划分为逻辑上独立的地址空间并且有助于共享和保护。**

## 6. 页面置换算法

在程序运行过程中，如果要访问的页面不在内存中，就发生**缺页中断**从而将该页调入内存中。此时如果内存已无空闲空间，系统必须从内存中调出一个页面到磁盘对换区中来腾出空间。

页面置换算法和**缓存淘汰策略**类似，可以将内存看成磁盘的缓存。在缓存系统中，缓存的大小有限，当有新的缓存到达时，需要淘汰一部分已经存在的缓存，这样才有空间存放新的缓存数据。

页面置换算法的主要目标是使页面置换频率最低（也可以说缺页率最低）。

1. **最佳（OPT, Optimal replacement algorithm）**

   所选择的被换出的页面将是最长时间内不再被访问，通常可以保证获得最低的缺页率。**是一种理论上的算法**，因为无法知道一个页面多长时间不再被访问。

   举例：一个系统为某进程分配了三个物理块，并有如下页面引用序列：

```html
7，0，1，2，0，3，0，4，2，3，0，3，2，1，2，0，1，7，0，1
```

开始运行时，先将 7, 0, 1 三个页面装入内存。当进程要访问页面 2 时，产生缺页中断，会将页面 7 换出，因为页面 7 再次被访问的时间最长。

2. **最近最久未使用（LRU, Least Recently Used）**

   虽然无法知道将来要使用的页面情况，但是可以知道过去使用页面的情况。LRU 将最近最久未使用的页面换出。

   为了实现 LRU，需要在内存中维护一个所有页面的链表。当一个页面被访问时，将这个页面移到链表表头。这样就能保证链表表尾的页面是最近最久未访问的。

   因为每次访问都需要更新链表，因此这种方式实现的 LRU 代价很高。

```html
4，7，0，7，1，0，1，2，1，2，6
```

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/eb859228-c0f2-4bce-910d-d9f76929352b.png"/> </div><br>

3. **最近未使用（NRU, Not Recently Used）**

   每个页面都有两个状态位：R 与 M，当页面被访问时设置页面的 R=1，当页面被修改时设置 M=1。其中 R 位会定时被清零。可以将页面分成以下四类：

    - R=0，M=0
    - R=0，M=1
    - R=1，M=0
    - R=1，M=1

    当发生缺页中断时，NRU 算法随机地从类编号最小的非空类中挑选一个页面将它换出。

    NRU 优先换出已经被修改的脏页面（R=0，M=1），而不是被频繁使用的干净页面（R=1，M=0）。

4. **先进先出（FIFO, First In First Out）**

   选择换出的页面是最先进入的页面。该算法会将那些经常被访问的页面换出，导致缺页率升高。

5. **第二次机会算法**

   FIFO 算法可能会把经常使用的页面置换出去，为了避免这一问题，对该算法做一个简单的修改：

   当页面被访问 (读或写) 时设置该页面的 R 位为 1。需要替换的时候，检查最老页面的 R 位。如果 R 位是 0，那么这个页面既老又没有被使用，可以立刻置换掉；如果是 1，就将 R 位清 0，并把该页面放到链表的尾端，修改它的装入时间使它就像刚装入的一样，然后继续从链表的头部开始搜索。

   <div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/ecf8ad5d-5403-48b9-b6e7-f2e20ffe8fca.png"/> </div><br>

6. **时钟Clock**

   第二次机会算法需要在链表中移动页面，降低了效率。时钟算法使用环形链表将页面连接起来，再使用一个指针指向最老的页面。

   <div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/5f5ef0b6-98ea-497c-a007-f6c55288eab1.png"/> </div><br>

# 五、设备管理

## 1. 磁盘结构

- 盘面（Platter）：一个磁盘有多个盘面；
- 磁道（Track）：盘面上的圆形带状区域，一个盘面可以有多个磁道；
- 扇区（Track Sector）：磁道上的一个弧段，一个磁道可以有多个扇区，它是最小的物理储存单位，目前主要有 512 bytes 与 4 K 两种大小；
- 磁头（Head）：与盘面非常接近，能够将盘面上的磁场转换为电信号（读），或者将电信号转换为盘面的磁场（写）；
- 制动手臂（Actuator arm）：用于在磁道之间移动磁头；
- 主轴（Spindle）：使整个盘面转动。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/014fbc4d-d873-4a12-b160-867ddaed9807.jpg"/> </div><br>

## 2. 磁盘调度算法

读写一个磁盘块的时间的影响因素有：

- 旋转时间（主轴转动盘面，使得磁头移动到适当的扇区上）
- 寻道时间（制动手臂移动，使得磁头移动到适当的磁道上）
- 实际的数据传输时间

其中，寻道时间最长，因此磁盘调度的主要目标是使磁盘的平均寻道时间最短。

1. **先来先服务（FCFS, First Come First Served）****

    按照磁盘请求的顺序进行调度。

    优点是公平和简单。缺点也很明显，因为未对寻道做任何优化，使平均寻道时间可能较长。

2. **最短寻道时间优先（SSTF, Shortest Seek Time First）**

    优先调度与当前磁头所在磁道距离最近的磁道。

    虽然平均寻道时间比较低，但是不够公平。如果新到达的磁道请求总是比一个在等待的磁道请求近，那么在等待的磁道请求会一直等待下去，也就是出现饥饿现象。具体来说，两端的磁道请求更容易出现饥饿现象。

    <div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/4e2485e4-34bd-4967-9f02-0c093b797aaa.png"/> </div><br>

3. **电梯算法（SCAN）**

    电梯总是保持一个方向运行，直到该方向没有请求为止，然后改变运行方向。

    电梯算法（扫描算法）和电梯的运行过程类似，总是按一个方向来进行磁盘调度，直到该方向上没有未完成的磁盘请求，然后改变方向。

    因为考虑了移动方向，因此所有的磁盘请求都会被满足，解决了 SSTF 的饥饿问题。

    <div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/271ce08f-c124-475f-b490-be44fedc6d2e.png"/> </div><br>

# 六、链接

## 1. 编译系统（了解一下）

1. 源文件

   源代码是相对目标代码和可执行代码而言的。 

   源代码就是用汇编语言和高级语言写出来的地代码。 

   源文件就是存放程序代码的文件，通常我们编辑代码的文件就是源文件，如.c、.cpp、.java、.asm等都是源文件

2. 目标文件

   目标代码是指源代码经过编译程序产生的能被cpu直接识别二进制代码。 

   目标文件由编译器生成，具体的生成方法在不同的开发环境上是不同的。

   目标代码文件包含着机器语言代码，它并不需要是完整的程序代码，如.o（.OBJ）

   例1：gcc -c test.c 

   编译test.c ,生成目标文件test.o，但不进行link

   例2：masm lab02.asm

   编译lab02.asm ,生成目标文件lab02.OBJ，但不进行link。

3. 可执行文件

   可执行文件包含了着组成可执行程序的全部机器语言代码。

   可执行代码就是将目标代码连接（link）后形成的可执行文件，连接程序系统库文件就生成可执行文件。

   常见的有.exe、.class、.com等

   例1：gcc -o test test.c 
   编译test.c生成可执行文件test 
   ./test
   运行可执行文件

   例2：link lab02.OBJ
   生成可执行文件lab02.EXE
   lab02
   运行可执行文件

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/b396d726-b75f-4a32-89a2-03a7b6e19f6f.jpg" width="800"/> </div><br>

**gcc 编译四个过程：**编译器：GCC，操作系统：linux系统

1、预处理过程（头文件的包涵，去掉注释，宏展开）—#include 预处理过程不做语法检查

命令：gcc -E helloworld.c -o helloworld.i

2、 编译：编译过程做语法检查 生成汇编语言

命令：gcc -S helloworld.i -o helloworld.s

3、汇编：将汇编语言生成对应的二进制数据即目标文件

命令：gcc -c helloworld.s -o helloworld.o

4、链接：添加对应操作系统可以执行的链接，得到可执行文件

命令：gcc helloworld.o -o helloworld

## 2. 静态链接

静态链接器以一组可重定位目标文件为输入，生成一个完全链接的可执行目标文件作为输出。链接器主要完成以下两个任务：

- 符号解析：每个符号对应于一个函数、一个全局变量或一个静态变量，符号解析的目的是将每个符号引用与一个符号定义关联起来。
- 重定位：链接器通过把每个符号定义与一个内存位置关联起来，然后修改所有对这些符号的引用，使得它们指向这个内存位置。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/47d98583-8bb0-45cc-812d-47eefa0a4a40.jpg"/> </div><br>

## 3. 动态链接

静态库有以下两个问题：

- 当静态库更新时那么整个程序都要重新进行链接；
- 对于 printf 这种标准函数库，如果每个程序都要有代码，这会极大浪费资源。

共享库是为了解决静态库的这两个问题而设计的，在 Linux 系统中通常用 .so 后缀来表示，Windows 系统上它们被称为 DLL。它具有以下特点：

- 在给定的文件系统中一个库只有一个文件，所有引用该库的可执行目标文件都共享这个文件，它不会被复制到引用它的可执行文件中；
- 在内存中，一个共享库的 .text 节（已编译程序的机器代码）的一个副本可以被不同的正在运行的进程共享。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/76dc7769-1aac-4888-9bea-064f1caa8e77.jpg"/> </div><br>

## 5. [一个程序从源代码到可执行程序的过程](https://blog.csdn.net/qq_39755395/article/details/78293733)（详细版）

一个源程序到一个可执行程序的过程：预编译、编译、汇编、链接。 其中，编译是主要部分，其中又分为六个部分：词法分析、语法分析、语义分析、中间代码生成、目标代码生成和优化。 链接中，分为静态链接和动态链接，本文主要是静态链接。

### （1）预编译：主要处理源代码文件中的以“#”开头的预编译指令

1.删除所有的#define，展开所有的宏定义。 2.处理所有的条件预编译指令，如“#if”、“#endif”、“#ifdef”、“#elif”和“#else”。 3.处理“#include”预编译指令，将文件内容替换到它的位置，这个过程是递归进行的，文件中包含其他文件。 4.删除所有的注释，“//”和“/**/”。 5.保留所有的#pragma 编译器指令，编译器需要用到他们，如：#pragma once 是为了防止有文件被重复引用。 6.添加行号和文件标识，便于编译时编译器产生调试用的行号信息，和编译时产生编译错误或警告是能够显示行号。

**C语言的宏替换和文件包含的工作，不归入编译器的范围，而是交给独立的预处理器。** C语言中源代码文件的文件扩展名为.c，头文件的文件扩展名为.h，经预编译之后，生成xxx.i文件。 在C++，源代码文件的扩展名是.cpp或.cxx，头文件的文件扩展名为.hpp，经预编译之后，生成xxx.ii文件。

### （2）编译：把预编译之后生成的xxx.i或xxx.ii文件

进行一系列词法分析、语法分析、语义分析及优化后，生成相应的汇编代码文件。(结合程序来说明编译的几个步骤) 有C语言的源代码如下： arr[3] = (a+4)*(3+8);

**1.词法分析：**利用类似于“有限状态机”的算法，将源代码程序输入到扫描机中，将其中的**字符序列分割成一系列的记号**。 以上的一行C语言程序，一共有16个空字符，经扫描机扫描之后，产生了16个记号。lex可以实现词法分析。见下表：

![这里写图片描述](https://img-blog.csdn.net/20171020114535621?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzk3NTUzOTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

见上图： 词法分析产生的**记号分类**有：关键字、标识符、字面量(数字、字符串)、特殊符号(加号、等号等)

**2.语法分析：**语法分析器对由扫描器产生的记号，进行语法分析，产生语法树。由语法分析器输出的***语法树是一种以表达式为节点的树\***。上述的代码就是 各种表达式的组合：赋值表达式、加法表达式、乘法表达式、数组表达式和括号表达式组成的复杂表达式。yacc可以实现语法分析，根据用户给定的规则(不同的编程语言对应不同的语法规则)对记号表进行解析。

![这里写图片描述](https://img-blog.csdn.net/20171020114014565?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzk3NTUzOTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

见上图： 整个语句被看作是一个“赋值表达式”，“=”左边是一个“数组表达式”，右边是一个“乘法表达式”。数组表达式又由两个符号表达式组成，符号表达式就是最小的表达式，之后同理。

**在语法分析的同时，就把运算符的优先级确定了下来，如果出现表达式不合法，——各种括号不匹配、表达式中缺少操作，编译器就会报错。**

**3.语义分析：**语法分析器只是完成了对表达式语法层面的分析，语义分析器则对**表达式是否有意义**进行判断，其分析的语义是静态语义——在编译期能分期的语义，相对应的动态语义是在运行期才能确定的语义。 其中，**静态语义通常包括：声明和类型的匹配，类型的转换**，那么语义分析就会对这些方面进行检查，例如将一个int型赋值给int*型时，**语义分析程序会发现这个类型不匹配，编译器就会报错。**

经过语义分析阶段之后，所有的符号都被标识了类型(如果有些类型需要做隐式转化，语义分析程序会在语法树中插入相应的转换节点)，见下图：

![这里写图片描述](https://img-blog.csdn.net/20171020114053319?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzk3NTUzOTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 这个语句中的类型都是int型，无须做转换。

**4.优化:**源代码级别的一个优化过程，例如该语句中的(3+8)的值可以在编译期确定，源代码优化器会将整个语法树转换成中间代码——语法树的顺序表示，十分接近目标代码。 中间代码有很多种类型，最常见的是“**三地址码**”和“P-代码”,其中三地址码的基本形式为：**x = y op z**，表示将变量y和z进行op操作后，赋值给x，op操作可以是加减乘除等。 经优化之后的语法树为：

![这里写图片描述](https://img-blog.csdn.net/20171020114156552?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzk3NTUzOTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

该语句的三地址码： t1 = 3 + 8; t2 = a + 4; t3 = t2 * t1; arr[3] = t3;

t1由数字11代替，省去t3，经优化或的三地址码为： t2 = a +4; t2 = t2 + 11; arr[3] = t2;

另一个关于中间代码的要点：***中间代码使得编译器可以被分成前端和后端\***，编译器前端负责产生与机器无关的中间代码，编译器后端将中间代码转换为机器代码。 源代码优化去产生中间代码标志着下面的过程都属于编译器后端，后端主要包括：代码生成器和目标代码优化器。

**5.目标代码生成：**由代码生成器将中间代码转换成目标机器代码，生成一系列的代码序列——汇编语言表示。

**6.目标代码优化：**目标代码优化器对上述的目标机器代码进行优化：寻找合适的寻址方式、使用位移来替代乘法运算、删除多余的指令等。

上述的六个步骤完毕之后，**编译过程**也就告一段落了。**最终产生了由汇编语言编写的目标代码**。

**gcc把预编译和编译两个步骤合并成一个步骤。**对于C语言的代码，是用“cc1”这个程序来完成这两步，对于C++代码，对应的程序为“cc1plus”。**gcc这个命令只是后台程序的包装**，根据不同的参数去调用：预编译编译程序——cc1，汇编器——as，连接器——ld。

C语言的代码，经编译后产生的文件名为xxx.s。

### （3）汇编：将汇编代码转变成机器可以执行的指令(机器码文件)。

汇编器的汇编过程相对于编译器来说更简单，没有复杂的语法，也没有语义，更不需要做指令优化，只是根据汇编指令和机器指令的对照表一一翻译过来，汇编过程有汇编器as完成。

经汇编之后，产生目标文件(与可执行文件格式几乎一样)xxx.o(Windows下)、xxx.obj(Linux下)。

但是，经过预编译、编译、汇编之后，生成机器可以执行的目标文件之后，还有**一个问题——变量a和数组arr的地址还没有确定**。这就需要链接器来搞定啦~

### （4）链接

**1、历史过程：**曾经，程序猿门在编程时，使用纸带作为最原始的存储设备，每当程序需要修改时，都要重新扎一条纸带，扎孔的表示1，不扎的是0，一串串1和0就组成了各种各样的指令——跳转等等…. 每一次的修改都非常痛苦，所以先知们就发明了汇编语言，这种编程语言方便之处在于符号的引用，表示跳转指令不再需要记住一串串0和1，终于可以使用符号——foo来表示这个动作了！ 随着汇编语言的普及，程序的代码量也就开始快速膨胀了，汇编语言说它也撑不住了….不过还好，高级编程语言Fortran、C、C++等一个接一个地问世，语言越来越方便了，追求perfect的人们就想：代码咋写更好呢？可不可以把代码按照功能的不同，分成不同的部分，便于日后的修改和重复使用呢？ 有了这个启发，程序猿们越来越得心应手，他们开始把代码按照功能和性质划分，分别形成不同的功能模块，不同的模块之间又按照各种结构来组织。 发展到如今，软件的规模越来越大，代码动辄数百万行代码，放在一个模块那是万万不行的，维护起来会非常麻烦，所有现在的大型软件往往拥有成千上万的模块， 模块之间相互独立又相互依赖。 新的问题来了，一个程序被分割成这么多模块，最后要怎么把这些模块组合形成一个单一的程序? 答案就是：***模块之间，符号的引用\***！ 这就像是一张画有大树的拼图，叶子、枝干、根系都零散的分布在那些拼图碎片上，想要看到完整的大树，我们就会耐心地把那些碎片拼合在一起。

![这里写图片描述](https://img-blog.csdn.net/20171020124207934?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzk3NTUzOTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这些模块之间同样如此，它们依靠那些凸起和凹陷联系在一起，最终组合成一个完整的程序，这样的过程称为——链接。

这样基于符号的模块化，使得链接过程在整个程序开发中显得十分重要和突出…..

**2、下面就静态链接，进行分析。** 1.链接：“组装”模块的过程。 2.链接的内容：把各个模块之间相互引用的部分都处理好，使得各个模块之间能够正确地衔接。(就像拼图，凸起和凹槽的位置一定一一对应，否则…) 3.链接的过程：地址和空间的分配、符号决议(也叫“符号绑定”，倾向于动态链接)和重定位 以gcc编译器为例，看基本的链接过程：

![这里写图片描述](https://img-blog.csdn.net/20171020122221125?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzk3NTUzOTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

.c文件经过编译器、汇编器之后得到目标文件.o，目标文件再与库进行链接得到可执行文件.out。 ***库其实就是一组目标文件的打包\***，这些目标文件中都是一些常用的代码。

我们在fun.c模块中定义了函数foo()，在main.c模块中引用了foo()函数，在编译过程当中，编译器并不知道main.c中foo()的地址，所以将调用foo()的指令的目标地址部分搁置， 等到了链接的阶段，链接器会去找到foo()定义的那个模块，在main.o中填入正确的函数地址，这个修改地址的过程被叫做“***重定位\***”，每个被修正的地方叫“重定位入口”。

![这里写图片描述](https://img-blog.csdn.net/20171020122343439?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzk3NTUzOTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

以上就是一个程序从源代码到可执行程序的大致过程，这是博主根据《程序员的自我修养——链接、装载与库》来整理的，有兴趣的同学可以自己去琢磨琢磨~

