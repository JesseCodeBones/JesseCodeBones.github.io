# Thread
## 线程共享和不共享的部分
除了全局内存之外，线程还共享了一干其他属性（这些属性对于进程而言是全局性的，
而并非针对某个特定线程），包括以下内容。
*  进程 ID（process ID）和父进程 ID。
*  进程组 ID 与会话 ID（session ID）。
* 控制终端。
* 进程凭证（process credential）（用户 ID 和组 ID ）。
* 打开的文件描述符。
由 fcntl()创建的记录锁（record lock）。

* 信号（signal）处置。
* 文件系统的相关信息：文件权限掩码（umask）、当前工作目录和根目录。
间隔定时器（setitimer()）和 POSIX 定时器（timer_create()）。
* 系统 V（system V）信号量撤销（undo，semadj）值（47.8 节）。
资源限制（resource limit）。
* CPU 时间消耗（由 times()返回）。
y资源消耗（由 getrusage()返回）。
nice 值（由 setpriority()和 nice()设置。  

各线程所独有的属性，如下列出了其中一部分。
*  线程 ID（thread ID，29.5 节）。
*  信号掩码（signal mask）。
* 线程特有数据（31.3 节）。
* 备选信号栈（sigaltstack()）。
errno 变量。
* 浮点型（floating-point）环境（见 fenv(3)）。
实时调度策略（real-time scheduling policy）和优先级（35.2 节和 35.3 节）。
* CPU 亲和力（affinity，Linux 所特有，35.4 节将加以描述）。
* 能力（capability，Linux 所特有，第 39 章将加以描述）。
* 栈，本地变量和函数的调用链接（linkage）信息。  

## 线程终止
* 线程 start 函数执行 return 语句并返回指定值。
* 线程调用 pthread_exit()（详见后述）。

* 调用 pthread_cancel()取消线程（在 32.1 节讨论）。
任意线程调用了 exit()，或者主线程执行了 return 语句（在 main()函数中）
，都会导致
进程中的所有线程立即终止。

## pthread_t
pthread_t 不可以作为一种不透明的数据类型加以对待，必须用pthread_equal()来进行比较。

## pthread_join
函数 pthread_join()等待由 thread 标识的线程终止, 如果线程已经终止，则方法立刻返回  

## 一个简单的多线程实例
```C++
pthread_t ptt;
  char arr[] = {"hello world\n"};
  void *ptr;
  void **returnval = &ptr;
  std::cout << "ptr" << std::hex << arr << std::endl;

  int s = pthread_create(
      &ptt, NULL,
      [](void *args) -> void * {
        std::cout << static_cast<char *>(args) << std::endl;
        return (void *)args;
      },
      arr);
  pthread_join(ptt, (void **)returnval);
  auto d = returnval[0];
  std::cout << "returned type from main thread-" << (char *)d << std::endl;
  return 0;
```
## pthread_create时的属性控制
pthread_create()中类型为 pthread_attr_t 的 attr 参数，其中包括：  
线程栈的位置和大小、线程调度策略和优先级  
示例： 
```C++
  pthread_t ptt;
  pthread_attr_t ptat;
  pthread_attr_init(&ptat);
  pthread_attr_setstacksize(&ptat, 10240);
  pthread_create(
      &ptt, &ptat,
      [](void *arg) -> void * { std::cout << "running in child thread\n"; },
      nullptr);
  pthread_join(ptt, nullptr);
  pthread_attr_destroy(&ptat);
```
## 总结
可使用 pthread_create()来创建线程。每个线程随后可调用 pthread_exit()独立退出。
（如有任一线
程调用了 exit()，那么所有线程将立即终止。
）除非将线程标记为分离状态（例如通过调用 pthread_
detached()）
，其他线程要连接该线程，则必须使用 pthread_join()，由其返回遭连接线程的退出状态

## 线程同步
临界区（critical section）是指访问某一共享资源的代码片段，并且这段代码的执行应为原子（atomic）操作，亦即，同时访问同一共享资源的其他线程不应中断该片段的执行。

一个经典的抢占临界区问题：
```C++
#include <iostream>
#include <thread>
static int grob;

int main(int, char **) {
  auto processFun = []() {
    for (int a = 0; a < 50000; ++a) {
      grob++;
    }
  };
  std::thread p1(processFun), p2(processFun);
  p1.join();
  p2.join();
  std::cout << "after process, grob = " << grob << std::endl;
}

```
一个简单的加锁实例
```C++
#include <iostream>
#include <mutex>
#include <thread>
static int grob;
static std::mutex m;
int main(int, char **) {
  auto processFun = []() {
    for (int a = 0; a < 50000; ++a) {
      std::lock_guard<std::mutex> guard(m);
      grob++;
    }
  };
  std::thread p1(processFun), p2(processFun);
  p1.join();
  p2.join();
  std::cout << "after process, grob = " << grob << std::endl;
}

```  
互斥量 mutex 加锁的原理：
* 调用 pthread_mutex_lock()时需要指定互斥量。如果互斥量当前处于未锁定状态，该调用将锁定互斥量并立即返回。
* 如果其他线程已经锁定了这一互斥量，那么 pthread_mutex_lock()调用会一直堵塞，直至该互斥量被解锁，到那时，调用将锁定互斥量并返回  
* 如果发起 pthread_mutex_lock()调用的线程自身之前已然将目标互斥量锁定可能会产生两种后果—视具体实现而定: 线程陷入死锁 或者 调用失败，返回 EDEADLK 错误。
对于通过pthread_mutex_unlock()

## 互斥量死锁问题
线程A:  
std::lock_guard<std::mutex> guard(m1);  
std::lock_guard<std::mutex> guard(m2);

线程B:  
std::lock_guard<std::mutex> guard(m2);  
std::lock_guard<std::mutex> guard(m1);

这样就很容易导致死锁。最简单的方法是定义互斥量的层级关系。
就是一个操作需要锁定两个互斥量的时候，要严格按照他们的顺序进行加锁。  
也可以使用`pthread_mutex_trylock()`进行尝试加锁，一旦调用失败，改线程将释放掉所有的互斥量，不过这种方法使用频率较低。

## Mutex的一些原则
* 同一线程不应对同一互斥量加锁两次。
* 线程不应对不为自己所拥有的互斥量解锁（亦即，尚未锁定互斥量）。
* 线程不应对一尚未锁定的互斥量做解锁动作。

## Mutex的类型
* PTHREAD_MUTEX_NORMAL 该类型的互斥量不具有死锁检测（自检）功能
* PTHREAD_MUTEX_ERRORCHECK 对此类互斥量的所有操作都会执行错误检查。所有上述 3 种情况都会导致相关 Pthreads 函数返回错误。这类互斥量运行起来比一般类型要慢，不过可将其作为调试工具，以发现程序在哪里违反了互斥量使用的基本原则。
* PTHREAD_MUTEX_RECURSIVE 递归互斥量维护有一个锁计数器。当线程第 1 次取得互斥量时，会将锁计数器置 1, 后续由同一线程执行的每次加锁操作会递增锁计数器的数值, 而解锁操作则递减计数器计数, 只有当锁计数器值降至 0 时，才会释放（release，亦即可为其他线程所用）该互斥量.

## 条件变量
Posix中条件变量的方法
函数 pthread_cond_signal()和 pthread_cond_broadcast()均可针对由参数 cond 所指定的条件变量而发送信号。pthread_cond_wait()函数将阻塞一线程，直至收到条件变量 cond 的通知。  
pthread_cond_signal()函数只保证唤醒至少一条遭到阻塞的线程，而 pthread_cond_broadcast()则会唤醒所有遭阻塞的线程，不过效率要低一些。  
pthread_cond_timedwait()与函数 pthread_cond_wait()几近相同，唯一的区别在于参数 abstime 来指定一个线程等待条件变量通知时休眠时间的上限。  


## 一个经典的producer consumer pattern
```C++
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>

const int MAX_ITEMS = 1000;

std::queue<int> items;
std::mutex itemsMutex;
std::condition_variable itemsCondition;

void producer() {
  for (int i = 0; i < MAX_ITEMS; i++) {
    std::unique_lock<std::mutex> lock(itemsMutex);
    itemsCondition.wait(lock, [](){ return items.size() < MAX_ITEMS; });
    items.push(i);
    std::cout << "product item: "<< i << std::endl;
    lock.unlock();
    itemsCondition.notify_one();
  }
}

void consumer() {
  for (int i = 0; i < MAX_ITEMS; i++) {
    std::unique_lock<std::mutex> lock(itemsMutex);
    itemsCondition.wait(lock, [](){ return !items.empty(); });
    int item = items.front();
    items.pop();
    std::cout << "Consumed item: " << item << std::endl;
    lock.unlock();
    itemsCondition.notify_one();
    
  }
}

int main() {
  std::thread producerThread(producer);
  std::thread consumerThread(consumer);
  producerThread.join();
  consumerThread.join();
  return 0;
}

```

