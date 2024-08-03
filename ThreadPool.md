![image](https://github.com/FelixQLL/InterviewRecord/assets/28554261/5ae18799-ab10-4215-a558-02161d121e05)

```C++
class ThreadPool {
 public:
  // 构造函数，传入线程数
  ThreadPool(size_t);
  template <class F, class... Args>
  auto enqueue(F&& f, Args&&... args)
      -> std::future<typename std::result_of<F(Args...)>::type>;
  ~ThreadPool();

 private:
  // 线程组
  std::vector<std::thread> workers;
  // 任务队列
  std::queue<std::function<void()> > tasks;

  // synchronization
  std::mutex queue_mutex; //互斥锁
  std::condition_variable condition;//条件变量
  bool stop;//停止标志
};

// the constructor just launches some amount of workers
inline ThreadPool::ThreadPool(size_t threads) : stop(false) {
  for (size_t i = 0; i < threads; ++i)
    //thread 禁用了拷贝构造和赋值运算，因此这边最好用emplace_back
    workers.emplace_back([this] {
      for (;;) {  //死循环的作用是一直让线程执行任务直至结束
        std::function<void()> task;

        {
          std::unique_lock<std::mutex> lock(this->queue_mutex);
          //线程将等待，直到线程池被停止或有新的任务
          this->condition.wait(
              lock, [this] { return this->stop || !this->tasks.empty(); });
          // 如果线程终止了 且 任务队列空了 函数结束
          if (this->stop && this->tasks.empty()) return;
          task = std::move(this->tasks.front());
          this->tasks.pop();
        }

        task();
      }
    });
}

// add new work item to the pool
template <class F, class... Args>
auto ThreadPool::enqueue(F&& f, Args&&... args)
    -> std::future<typename std::result_of<F(Args...)>::type> {
  using return_type = typename std::result_of<F(Args...)>::type;

  auto task = std::make_shared<std::packaged_task<return_type()> >(
      std::bind(std::forward<F>(f), std::forward<Args>(args)...));

  std::future<return_type> res = task->get_future();
  {
    std::unique_lock<std::mutex> lock(queue_mutex);

    // don't allow enqueueing after stopping the pool
    if (stop) throw std::runtime_error("enqueue on stopped ThreadPool");

    tasks.emplace([task]() { (*task)(); }); // 将任务加入任务队列
  }
  condition.notify_one(); // 通知线程
  return res;
}

// the destructor joins all threads
inline ThreadPool::~ThreadPool() {
  {
    std::unique_lock<std::mutex> lock(queue_mutex);
    stop = true;
  }
  condition.notify_all();
  for (std::thread& worker : workers) worker.join();
}

```
# 什么是线程池？线程池有什么好处？
所谓线程池，通俗来讲，就是一个管理线程的池子。它可以容纳多个线程，其中的线程可以反复利用，省去了频繁创建线程对象的操作。
线程池的原理就是管理一个任务队列和一个工作线程队列。工作线程不断的从任务队列取任务，然后执行。如果没有任务就等待新任务的到来。添加新任务的时候先添加到任务队列，然后通知任意(条件变量notify_one/notify_all)一个线程有新的任务来了。

好处：

**资源管理：** 线程池有效地管理线程的创建、销毁和重用，避免了频繁创建和销毁线程的开销，节省了系统资源。

**减少线程创建时间：** 线程创建和销毁是开销较大的操作。线程池在初始化时创建一组线程，并将它们保持在就绪状态，从而在需要时可以快速执行任务，而不必每次都重新创建线程。

**任务队列：** 线程池通常与任务队列结合使用，任务可以被提交到队列中，线程池中的线程会按照队列中任务的顺序依次执行，确保了任务的有序执行。

**限制并发数：** 线程池可以限制并发执行的任务数量，以避免系统资源过度占用，提高系统稳定性。

**节省内存：** 线程池的线程可以被重复使用，避免了频繁创建线程的内存占用。

# 应用场景包括
**服务器应用：** 线程池在服务器应用中常用来处理客户端请求。当服务器需要处理大量的连接请求时，线程池可以有效地复用线程，提高服务器性能。

**I/O密集型任务：** 线程池适用于I/O密集型任务，如文件读写、网络通信等。在这些情况下，线程可以在I/O操作等待时执行其他任务，提高了系统的效率。

**并行计算：** 线程池可以用于并行计算，将任务拆分为多个子任务，由线程池中的线程并行执行，加速计算过程。

**定时任务：** 线程池可用于执行定时任务，例如定期备份数据、清理日志等。

**多任务并行处理：** 当需要处理多个任务，但不想为每个任务创建一个线程时，线程池是一种有效的方式。它可以控制并发任务的数量，从而避免系统过度负载。

# 互斥锁 mutex
一种同步原语，用于多线程管理，确保临界资源只有一个线程访问，其中mtx.lock()表示上锁，mtx.unlock()表示解锁。**上述线程池中为什么没有unlock呢？因 std::unique_lock<std::mutex> 这个智能指针在退出{}代码块之后就会自动销毁。**

# condition_variable
condition_variable是 C++ 标准库提供的一个同步原语，用于在线程之间进行等待和通知。
**等待：** 使线程等待，直到某个条件满足。等待期间，线程会释放持有的互斥锁，以便其他线程能够修改条件。例如：线程将等待，直到线程池被停止（this->stop为true）或有新的任务（!this->tasks.empty()）
**通知：** 唤醒等待的线程，通知它们某个条件已经改变。通知可以是单个线程（notify_one）或所有等待线程（notify_all）。

**避免虚假唤醒：** 使用带谓词的 wait 函数来避免虚假唤醒。谓词确保线程在被唤醒时再次检查条件是否满足。
```
cv.wait(lock, [] { return !dataQueue.empty() || finished; });
```

# 线程池如何确定线程的个数
**CPU密集型：** 最佳线程数等于cpu核心数或稍微小于cpu核心数  Cpu的核数 = 线程数就行，一般我们会设置 Cpu核数+1 防止由于其他因素导致线程阻塞等。
**耗时io型：** 最佳线程数一般会大于cpu核心数很多倍。。一般是io设备延时除以cpu处理延时，得到一个倍数，我的经验数值是20--50倍*cpu核心数。
多核Cpu 最佳线程数 =CPU 核数 * [ 1 +（I/O 耗时 / Cpu 耗时）

# 上述代码实现缺点
线程在初始化时全部创建，析构时销毁，没有超时回收等逻辑。

**优化方向：**

**动态线程管理：** 根据任务负载动态创建和回收线程，保证线程池中的线程数量在 minThreads 到 maxThreads 之间。

**空闲超时回收：** 如果一个线程在指定的 idleTimeout 时间内没有接收到任务，并且线程池中线程数量超过最小线程数，该线程将被回收。

``` C++
#ifndef THREADPOOL_H
#define THREADPOOL_H

#include <vector>
#include <queue>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <functional>
#include <future>
#include <chrono>

class ThreadPool {
 public:
  ThreadPool(size_t minThreads, size_t maxThreads, std::chrono::seconds idleTimeout);
  template <class F, class... Args>
  auto enqueue(F&& f, Args&&... args)
      -> std::future<typename std::result_of<F(Args...)>::type>;
  ~ThreadPool();

 private:
  void workerThread();

  std::vector<std::thread> workers;
  std::queue<std::function<void()> > tasks;
  std::mutex queue_mutex;
  std::condition_variable condition;
  bool stop;

  size_t minThreads;
  size_t maxThreads;
  std::chrono::seconds idleTimeout;
  size_t idleCount;  // Number of idle threads
};

inline ThreadPool::ThreadPool(size_t minThreads, size_t maxThreads, std::chrono::seconds idleTimeout)
    : stop(false), minThreads(minThreads), maxThreads(maxThreads), idleTimeout(idleTimeout), idleCount(0) {
  for (size_t i = 0; i < minThreads; ++i)
    workers.emplace_back([this] { workerThread(); });
}

template <class F, class... Args>
auto ThreadPool::enqueue(F&& f, Args&&... args)
    -> std::future<typename std::result_of<F(Args...)>::type> {
  using return_type = typename std::result_of<F(Args...)>::type;

  auto task = std::make_shared<std::packaged_task<return_type()> >(
      std::bind(std::forward<F>(f), std::forward<Args>(args)...));

  std::future<return_type> res = task->get_future();
  {
    std::unique_lock<std::mutex> lock(queue_mutex);

    if (stop) throw std::runtime_error("enqueue on stopped ThreadPool");

    tasks.emplace([task]() { (*task)(); });

    if (idleCount == 0 && workers.size() < maxThreads) {
      workers.emplace_back([this] { workerThread(); });
    }
  }
  condition.notify_one();
  return res;
}

void ThreadPool::workerThread() {
  while (true) {
    std::function<void()> task;
    {
      std::unique_lock<std::mutex> lock(this->queue_mutex);
      ++idleCount;
      if (this->condition.wait_for(lock, idleTimeout, [this] { return this->stop || !this->tasks.empty(); })) {
        --idleCount;
        if (this->stop && this->tasks.empty()) return;
        task = std::move(this->tasks.front());
        this->tasks.pop();
      } else {
        --idleCount;
        if (workers.size() > minThreads) {
          auto it = std::find_if(workers.begin(), workers.end(),
                                 [](const std::thread &t) { return !t.joinable(); });
          if (it != workers.end()) {
            it->detach();
            workers.erase(it);
            return;
          }
        }
      }
    }

    if (task) task();
  }
}

inline ThreadPool::~ThreadPool() {
  {
    std::unique_lock<std::mutex> lock(queue_mutex);
    stop = true;
  }
  condition.notify_all();
  for (std::thread& worker : workers) worker.join();
}

#endif
```
