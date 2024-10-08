# 什么是进程（Process）
**进程**是操作系统中最基本的并发执行单位。每个进程都有自己独立的内存空间，包含代码、数据、文件描述符等资源。
  - 进程拥有代码和打开的文件资源、数据资源、独立的内存空间
  - 进程是**操作系统进行资源分配和调度的基本单位**
  - 进程有五种状态：创建、就绪、阻塞、运行、关闭，五种状态的转换关系图为：
    ![image](https://github.com/FelixQLL/InterviewRecord/assets/28554261/78ce4860-1c2e-4afc-9034-11f804f82408)

# 进程间通信方式
1. 管道：半双工；数据只能单向流动，只能用于具有亲缘关系的进程之间，即用于父子、兄弟之间。
2. 命名管道（FIFO）：半双工，允许无亲缘关系的进程
3. 消息队列：消息链表存于内核，每个消息队列由消息队列标识符标识；于管道不同的是，消息队列存放在内核中，只有在内核重启时才能删除一个消息队列；消息队列的大小受限制。
4. 信号量（semophore）：信号量是一个计数器，可以用来控制多个进程对于共享资源的访问。作为一种锁机制，防止某进程正在访问共享资源师，其他进程也访问该资源，常用来处理临界资源的访问同步问题。
   临界资源：为某一时刻只能由一个进程或线程操作的资源。
5. 共享内存：就是映射一段能被其他进程所访问的内存，这段内存由一个进程创建，但可以多个进程同时访问，可以说是最有用的进程间通信方式，也是最快的IPC形式。常与其他通讯机制（信号量）配合使用。
6. 套接字（Socket）：适用于本地和网络通信，提供了强大的双向通信能力，也可用于不同机器之间。
7. 信号（Signal）：比较复杂，用于通知接收进程某个事件已经发生

**进程间通信：**
- **管道模型：** 比如cat xx.txt | grep -n 'xxx' 这个'|'可以看作一个单向的匿名管道，使得前后两个进程之间进行通信，前一个命名的输出作为后一个命令的输入，该管道使用结束则立即销毁。命名管道在linux中以文件的形式存在，只要访问该文件就可以实现任意两个进程间的通信，命名管道可以看作是是硬盘上存在的设备文件，所以打开需要使用open。
- **消息队列模型：** 比如生产者消费者模式，一端生产一端消费数据。遵循严格的先进先出。
- **共享内存：** 多用于传输一些大文件，如果采用管道或者消息队列传输大文件，涉及到重复拷贝，比较消耗性能，因此模拟多线程，在内存中开辟一块特殊的内存用于多个进程共享访问。也是进程间最高效的通信方式。
- **信号量机制：** 信号量可以看作一种数据操作锁，通过对临界资源的控制访问以管理进程之间的通信，PV原语操作。
- **socket：** 用于多个进程之间的网络传输。可以是单机多进程也可以是不同机器上的多进程通信。打开的socket在linux下也是以文件描述符fd存在。服务端创建套接字，绑定ip端口，监听端口号，等待客户端调用，客户端创建之后与服务端TCP三次握手建立连接完成，双方就可以发送和接收数据。

# 进程和CPU的关系
- **进程调度：** 操作系统通过进程调度算法（如先来先服务、短作业优先、优先级调度、时间片轮转）决定哪个进程在何时获得CPU时间。调度器是操作系统中负责选择和切换进程的组件。
- **上下文切换：** 上下文切换是操作系统在不同进程之间切换CPU执行权的过程。此过程涉及保存当前进程的状态（如寄存器内容、程序计数器等）并恢复下一个进程的状态。上下文切换虽然必要，但代价较高。
- **多任务处理：** 多任务处理让多个进程“并发”运行。通过快速的上下文切换，CPU能在多个进程之间分时执行，使每个进程看起来都在同时运行。多核CPU可以同时执行多个进程或线程，每个核心相当于一个独立的处理单元。
- **资源分配和管理：** 操作系统负责分配CPU时间片、内存、I/O设备等资源给各个进程。进程调度程序和内存管理程序共同工作，确保资源有效利用和进程之间的公平竞争。

# 什么是线程（Thread）
**线程**是进程中的一个执行单元，一个进程可以包含多个线程。线程共享进程的内存空间和资源，因此线程之间的通信相对简单，但也容易引发并发问题，如数据竞争。
  - 线程拥有自己的栈空间
  - 它可与同属一个进程的其他的线程**共享**进程所拥有的全部资源
  - 线程是最小的执行单元
  - 无论进程还是线程，都是由操作系统所管理的

# 线程有哪几种状态
新建（new） - 就绪（ready） - 运行（running） - 阻塞 （blocked） - 终止（terminated）

# 线程的状态切换
- **新建 → 就绪：** 当线程被创建并调用启动函数后，线程会进入就绪状态，等待被操作系统调度执行。
- **就绪 → 运行：** 操作系统使用某种调度算法（如先进先出、优先级调度等）从就绪队列中选择一个线程，将其状态从就绪状态切换为运行状态，并在CPU上执行其代码。
- **运行 → 阻塞：** 当线程执行到某个阻塞点时（如：执行了Thread.sleep(int n)方法等待I/O操作完成、获取某个锁等），线程会主动放弃对CPU的使用权，进入阻塞状态。操作系统会将线程从运行状态中移除，并将其添加到相应的阻塞队列中。
- **阻塞 → 就绪：** 当阻塞的原因消除后（如I/O操作完成、锁被释放等），操作系统会将线程从阻塞队列中移除，并将其状态切换为就绪状态，等待被调度执行。
- **运行 → 终止：** 当线程执行完毕或因异常而终止时，线程会进入死亡状态。操作系统会回收线程的资源，并将线程从所有队列中移除。

# 线程同步的方式
1. **互斥锁（Mutex）：** 提供了以排他方式防止数据结构被并发修改的方法，互斥对象和临界区对象非常相似，只是其允许在进程间使用，也可在线程间使用，而临界区只限制与同一进程的各个线程之间使用。
2. **条件变量（Condition Variable）**：以原子的方式阻塞进程，直到某个特定条件为真为止，一个线程被挂起，直到某件事件发生。条件变量始终与互斥锁一起使用。
3. **信号量（semaphore）：** 信号量是一种更复杂的同步原语，它可以用来控制多个线程对共享资源的访问。信号量维护一个计数器，表示可用资源的数量，线程可以通过信号量进行等待或释放。mutex是semaphore的一种特殊情况（n=1时）。也就是说，完全可以用后者替代前者。但是，因为mutex较为简单，且效率高，所以在必须保证资源独占的情况下，还是采用这种设计。
``` C++
#include <semaphore.h>

sem_t semaphore;

void thread1() {
    sem_wait(&semaphore); // 等待信号量
    // 访问共享资源
    sem_post(&semaphore); // 释放信号量
}

void thread2() {
    sem_post(&semaphore); // 发送信号量
}
```  
4. 原子操作

# 如何进行线程切换的
线程切换是操作系统的调度器通过保存当前线程的状态到线程的上下文中，然后加载另一个线程的上下文并恢复其状态，继续执行新线程的处理

# 线程切换需要保存的上下文，保存在哪里
保存的上下文包括：寄存器状态，程序计数器，堆栈指针等。这些信息通常保存在系统内存中，具体位置由操作系统决定，通常是在对应线程的内核栈或者线程的控制块中。

# 互斥锁和自旋锁的区别
- **实现原理：** 互斥锁依赖于操作系统提供的原语或系统调用，自旋锁依赖于硬件提供的原子操作
- **阻塞方式：** 互斥锁会导致线程阻塞和唤醒，而自旋锁会在循环中等待。
- **适用场景：** 互斥锁适用于长期占用临界资源的情况，而自旋锁适用于短期占用临界资源的情况。
- **开销：**
    - 互斥锁在获取锁时会导致线程阻塞，线程会被放入阻塞队列中，并在锁释放时被唤醒。这会引起线程上下文切换的开销。
    - 自旋锁在获取锁时会循环检查锁的状态，直到获取到锁为止，期间线程会一直占用CPU资源，但不会进入阻塞状态，也不会加入到阻塞队列中，开销更小

# 自旋锁等待时线程处于什么状态，互斥锁呢
自旋锁等待是线程处于**运行态**，互斥锁阻塞时线程处于 **阻塞态**

# 刚拿到互斥锁的线程处于什么状态
**阻塞态 -> 就绪态 -> 运行态**
当持有互斥锁的线程释放锁后，等待该锁的线程会被唤醒，进入**就绪状态**。这意味着线程准备好运行，等待 CPU 调度。
一旦调度器选择该线程并分配 CPU 时间，它就会从**就绪状态**转换到**运行状态**。此时，线程已成功获取互斥锁，并开始执行锁后面的代码。

# 什么是协程（Coroutine）
**协程**是一种更轻量级的并发机制，可以在单线程中实现类似多线程的并发效果。一个线程可以拥有多个协程。
**协程**是一种可以暂停和恢复执行的函数。

# 协程的上下文切换
**步骤：**
1. 保存当前执行上下文：保存当前寄存器和堆栈指针
2. 加载目标执行上下文
3. 切换到目标上下文

# 进程与线程区别
1. **内存分配**：
   - 进程：每个进程有自己独立的地址空间，一个进程崩溃后，在保护模式下不会影响其他进程。
   - 线程：所有线程共享其母进程的地址空间，一个线程的错误可能会导致整个进程的崩溃。
2. **通信方式**：
   - 进程：进程间通信需要特殊的IPC机制。
   - 线程：线程间可以直接读写进程数据段（如全局变量）来进行通信。
3. **系统开销**：
   - 进程：进程在创建、销毁、切换时的系统开销大，因为涉及到对应的地址空间的创建和销毁。
   - 线程：线程的创建、销毁、切换的开销相对较小，因为它们共享很多资源。
4. **资源管理**：
   - 进程：作为资源分配的基本单位，独立进程拥有完整资源集合。
   - 线程：作为调度执行的基本单位，只维护必要的信息和资源供运行。
5. **依赖关系**：
   - 进程：进程可以独立执行，不依赖其他进程。
   - 线程：线程是进程的一部分，依赖于进程的存在。
6. **执行环境**：
   - 进程：每个进程提供给其内部线程的执行环境相对复杂。
   - 线程：线程拥有较为简单的执行环境。

# 协程和线程的区别
![image](https://github.com/FelixQLL/InterviewRecord/assets/28554261/64382068-c123-4b97-aeb5-953fdc3f80c6)


# ROS 通信机制
参考：https://blog.csdn.net/qq_66257231/article/details/125023331

1. **Ros Topic：**
   - 异步通信：发布者和订阅者独立运行，不需要彼此等待。
   - 多对多通信：一个 topic 可以有多个发布者和多个订阅者。
   - 消息中介：在ROS中，ROS Master管理各个节点的注册信息，确保发布者和订阅者能够找到彼此，但消息的实际传递由点对点（peer-to-peer）连接完成。
2. **Ros Service：**
   - 同步通信：客户端发送请求后等待服务端的响应，客户端和服务端之间存在同步等待关系。
   - 一对一通信：每个服务只有一个提供者（服务端），但可以有多个请求者（客户端）。
   - 直接连接：客户端和服务端之间建立直接的连接来传递请求和响应。
3. **Ros Param：**
   - 集中式存储：参数服务器集中存储所有参数，允许各个节点访问。
   - 键值对存储：参数以键值对的形式存储，每个参数都有一个唯一的键。
   - 支持多种数据类型：参数可以是整数、浮点数、字符串、布尔值、列表和字典等。
   - 生命周期与 ROS Master 一致：参数服务器与 ROS Master 共享生命周期，当 ROS Master 关闭时，参数服务器也会关闭。
##### 总结：
**通信过程：发布者/订阅者（客户端/服务端）在Master注册 -> Master根据name匹配 -> 发布者/订阅者（客户端/服务端）建立TCP/UDP网络连接 -> 通信**
ROS通信基于TCP/UDP，ROS Param 不涉及TCP/UDP通信

# ROS Topic 和 Service的区别
![image](https://github.com/FelixQLL/InterviewRecord/assets/28554261/177778ce-081a-4040-9d28-c055d5e9f83d)

# TCP/UDP 区别
![image](https://github.com/FelixQLL/InterviewRecord/assets/28554261/899eb879-ebf8-4810-8796-05aa1d4d621f)
1. TCP传输数据前必须建立好连接，UDP无连接
2. TCP一对一服务，UDP支持一对一，一对多，多对一，多对多
3. TCP是可靠交付，UDP尽最大努力交付，不保证可靠
4. TCP有拥塞控制和流量控制，保证数据传输的安全性；UDP没有
5. 报文长度：
   - TCP动态报文长度，根据接收方窗口大小和网络拥塞情况决定
   - UDP不合并，不拆分
6. 首部开销：
   - TCP 20字节
   - UDP 8字节

**适用场景：**
  - TCP 可靠，传输速度慢，适用文件传输、状态更新
  - UDP不可靠，传输速度快，适用视频传输，实时通信

# TCP如何保证可靠性
1. 序列号，确认应答，超时重传
2. 窗口控制与高速重发控制/快速重传：再一个窗口大小内，不用等到应答就可以发送下一段数据
3. 拥塞控制

# TCP 拥塞控制
1. 慢开始：最开始发送方拥塞窗口为1，由小到大逐渐增大发送窗口和拥窗口，每经过一个传输伦次，cwnd加倍，cwnd超过慢开始门限，使用拥塞避免算法，避免cwnd增长过大
2. 拥塞避免：一旦发现网络拥塞，就把慢开始门限设为当前值的一半，重新设置cwnd为1，重新慢启动
3. 快重传：接收方每次收到一个失序的报文段立即发出重复确认，发送方连续收到三个重复确认就立即重传
4. 快恢复：发送方连续收到三个重复确认，就乘法减半（慢开始门限减半），将当前的cwnd设置为慢开始门限，采用拥塞避免算法
