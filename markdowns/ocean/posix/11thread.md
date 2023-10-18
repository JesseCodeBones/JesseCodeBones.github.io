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

## 互斥锁和自旋锁的区别
自旋锁是一种基础的同步原语，用于保障对共享数据的互斥访问。与互斥锁的相比，在获取锁失败的时候不会使得线程阻塞而是一直自旋尝试获取锁。当线程等待自旋锁的时候，CPU不能做其他事情，而是一直处于轮询忙等的状态。自旋锁主要适用于被持有时间短，线程不希望在重新调度上花过多时间的情况。实际上许多其他类型的锁在底层使用了自旋锁实现，例如多数互斥锁在试图获取锁的时候会先自旋一小段时间，然后才会休眠。  

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

## 造成函数线程不安全的原因
* 使用了在所有线程之间共享的全局或静态变量  
解决方法： 将共享变量与互斥量关联起来

不可重入函数的原因
* 根据其性质，有些函数必须访问全局数据结构。malloc 函数库中的函数就是这方面的典
范。这些函数为堆中的空闲块维护有一个全局链表。malloc 库函数的线程安全是通过使
用互斥量来实现的。
* 一些函数（在发明线程之前就已问世）的接口本身就定义为不可重入，要么返回指
针，指向由函数自身静态分配的存储空间，要么利用静态存储对该函数（或相关函
数）历次调用间的信息加以维护。表 31-1 所列函数大多属于此类。例如，函数 asctime()
（10.2.3 节）就返回一个指针，指向经由静态分配的缓冲区，其内容为日期和时间字
符串。

对于一些接口不可重入的函数，SUSv3 为其定义了以后缀_r 结尾的可重入“替身”。这些“替身”函数要求由调用者来分配缓冲区，并将缓存区地址传给函数用以返回结果。如：：asctime_r(), ctime_r()、getgrgid_r()、getgrnam_r()、getlogin_r()、getpwnam_r()、getpwuid_r()、gmtime_r()，localtime_r()、rand_r()、readdir_r()、strerror_r()、strtok_r()和 ttyname_r()。  

## pthread_once
函数pthread_once()可以确保无论有多少线程对也只会执行一次由 init 指向的调用者定义函数  

## 线程特有数据
使用线程特有数据技术，可以无需修改函数接口而实现已有函数的线程安全。较之于可重入函数，采用线程特有数据的函数效率可能要略低一些，不过对于使用了这些调用的程序而言，则省去了修改程序之劳。  
线程特有数据使函
数得以为每个调用线程分别维护一份变
量的副本（copy）。线程特有数据是长期 图 31-1：线程特有数据（TSD）为函数提供线程内存储
存在的。在同一线程对相同函数的历次调用间，每个线程的变量会持续存在，函数可以向每
个调用线程返回各自的结果缓冲区（如果需要的话）。

### 创建线程特有数据的步骤：  
1． 函数创建一个键（key），用以将不同函数使用的线程特有数据项区分开来。调用函数
pthread_key_create()可创建此“键”，且只需在首个调用该函数的线程中创建一次，函数
pthread_once()的使用正是出于这一目的。键在创建时并未分配任何线程特有数据块。
2． 调用 pthread_key_create()还有另一个目的，即允许调用者指定一个自定义解构函数，用于
释放为该键所分配的各个存储块（参见下一步）。当使用线程特有数据的线程终止时，
Pthreads API 会自动调用此解构函数，同时将该线程的数据块指针作为参数传入。
3． 函数会为每个调用者线程创建线程特有数据块。这一分配通过调用 malloc()（或类似函数）
完成，每个线程只分配一次，且只会在线程初次调用此函数时分配。
4． 为了保存上一步所分配存储块的地址，函数会使用两个 Pthreads 函数：pthread_setspecific()和
pthread_getspecific()。调用函数 pthread_setspecific()实际上是对 Pthreads 实现发起这样的请
求：保存该指针，并记录其与特定键（该函数的键）以及特定线程（调用者线程）的关联
性。调用 pthread_getspecific()所执行的是互补操作：返回之前所保存的、与给定键以及调
用线程相关联的指针。如果还没有指针与特定的键及线程相关联，那么 pthread_getspecific()
返回 NULL。函数可以利用这一点来判断自身是否是初次为某个线程所调用，若为初次，
则必须为该线程分配空间。

`pthread_key_create(pthread_key_t* key, void(*destructor)(void *));`
只要线程终止时与 key 的关联值不为 NULL，Pthreads API 会自动执行解构函数并将与key 的关联值作为参数传入解构函数。  

### 主线程中处理子线程的exception
下面的代码是不可以的
```C++
int main() {
  std::thread thread([]() {
    for (unsigned int i = 0U; i < 3U; ++i) {
      std::cout << std::to_string(i) << std::endl;
      std::this_thread::sleep_for(std::chrono::seconds(1U));
    }
    throw std::runtime_error("test");
  });
  std::this_thread::sleep_for(std::chrono::seconds(4U));
  std::cout << "sleep finished\n"; // this will not be printed
  try {
    thread.join();
  } catch (...) {
    std::cout << "error happened during child thread\n";
  }
  std::cout << "run finished\n";
}
```
下面是准确的做法
```C++
int main() {
  std::promise<std::exception_ptr> promise;
  std::thread thread([&promise]() {
    for (unsigned int i = 0U; i < 3U; ++i) {
      std::cout << std::to_string(i) << std::endl;
      std::this_thread::sleep_for(std::chrono::seconds(1U));
    }
    promise.set_exception(std::make_exception_ptr(
        std::runtime_error("test child thread exception")));
  });
  std::this_thread::sleep_for(std::chrono::seconds(4U));
  std::cout << "sleep finished\n";
  try {
    thread.join();
    promise.get_future().get();
  } catch (std::runtime_error e) {
    std::cout << e.what() << std::endl;
  }
  std::cout << "run finished\n";
}
```

### 线程对信号的过滤
信号掩码（mask）是针对每个线程而言。（对于多线程程序来说，并不存在一个作用
于整个进程范围的信号掩码，可以管理所有线程。）使用 Pthreads API 所定义的函数
pthread_sigmask()，各线程可独立阻止或放行各种信号。通过操作每个线程的信号掩
码，应用程序可以控制哪些线程可以处理进程收到的信号

### 取消一个线程

pthread_cancel和pthread_join（清理线程）
```C++
#include <pthread.h>
#include <unistd.h>

static void* threadFun(void* args){
  int j = 0;
  printf("New thread start:\n");
  for(j=0;;j++){
    printf("loop print: %d\n", j);
    sleep(1);
  }
  return nullptr;
}

int main() {
  pthread_t pth;
  int s;
  void *res;
  s = pthread_create(&pth, NULL, threadFun, NULL);
  if (s!=0) {
    exit(1);
  }
  sleep(3);
  s = pthread_cancel(pth);
  pthread_join(pth, &res);
  if (res == PTHREAD_CANCELED) {
    printf("thread canceled successfully\n");
  } else {
    printf("thread cannot be canceled\n");
  }
}
```
因为线程的取消需要线程运行到可取消点函数，如果没有执行到，线程可能永远不会被取消。  
可以共过`pthread_testcancel()`来检测线程是否是可以被取消的。  

### 进程取消时的清理工作
线程在执行到取消点时如果只是草草收场，这会将共享
变量以及 Pthreads 对象（例如互斥量）置于一种不一致状态，可能导致进程中其他线程产生错
误结果、死锁，甚至造成程序崩溃。为规避这一问题，线程可以设置一个或多个清理函数，当线
程遭取消时会自动运行这些函数，在线程终止之前可执行诸如修改全局变量，解锁互斥量等动作。

函数 pthread_cleanup_push()和 pthread_cleanup_pop()分别负责向调用线程的清理函数栈
添加和移除清理函数。

下面是一个线程可以被取消并且正确清理其中的资源的多线程例子：  
```C++
#include <cstddef>
#include <pthread.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

static pthread_mutex_t _mutex;
static pthread_cond_t _cond;
static int _global;

static void cleanUp(void* args) {

  if(args) {
    free(args);
  }
  pthread_mutex_unlock(&_mutex);
}

static void* thread_fun(void* args) {
  pthread_cleanup_push(cleanUp, nullptr);
  for (int i = 0; i < 100000; ++i) {
    pthread_mutex_lock(&_mutex);
    _global++;
    pthread_testcancel(); // cancel point, almost no cost
    if(i%10000==0) {
      sleep(2);
    }
    pthread_mutex_unlock(&_mutex);
  }
  pthread_cleanup_pop(1);
  return nullptr;
}

int main(){

  pthread_t p1, p2;
  void * res;
  pthread_create(&p1, NULL, thread_fun, NULL);
  pthread_create(&p2, NULL, thread_fun, NULL);
  sleep(7);
  pthread_cancel(p1);
  pthread_join(p1, &res);
  if(res == PTHREAD_CANCELED) {
    printf("p1 canceled successfully\n");
  }
  printf("target: %d\n", _global);
}
```

### 多线程中调用多进程的问题
当多线程进程调用 fork()时，仅会将发起调用的线程复制到子进程中。（子进程中该线程
的线程 ID 与父进程中发起 fork()调用线程的线程 ID 相一致。）其他线程均在子进程中消失，
也不会为这些线程调用清理函数以及针对线程特有数据的解构函数。

只要有任一线程调用了 exec()系列函数之一时，调用程序将被完全替换。除了调用 exec()的线
程之外，其他所有线程都将立即消失。没有任何线程会针对线程特有数据执行解构函数
（destructor），也不会调用清理函数（cleanup handler）。该进程的所有互斥量（为进程私有）和
属于进程的条件变量都会消失。调用 exec()之后，调用线程的线程 ID 是不确定的。


不要将线程与信号混合使用，只要可能多线程应用程序的设计应该避免使用信号。如果
多线程应用必须处理异步信号的话，通常最简洁的方法是所有的线程都阻塞信号，创建一个
专门的线程调用 sigwait()函数（或者类似的函数）来接收收到的信号。这个线程就可以安全地
执行像修改共享内存（处于互斥量的保护之下）和调用非异步信号安全的函数


正确多线程中处理信号的例子：
```C++
#include <bits/types/sigset_t.h>
#include <csignal>
#include <cstddef>
#include <iostream>
#include <pthread.h>
#include <signal.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void *sigHandlerThreadFun(void *args) {
  sigset_t set;
  int sig;
  sigemptyset(&set);
  sigaddset(&set, SIGTSTP);
  pthread_sigmask(SIG_BLOCK, &set, nullptr);
  printf("waiting... sigtstp\n");
  while (true) {
    sigwait(&set, &sig); // 子线程等待SIGTSTP然后处理
    printf("got it sigtstp\n");
  }
  return nullptr;
}

int main() {

  // 主线程中碰到信号，直接阻塞掉，不去处理
  sigset_t set;
  sigemptyset(&set);
  sigaddset(&set, SIGTSTP);
  sigaddset(&set, SIGUSR1);
  pthread_sigmask(SIG_BLOCK, &set, NULL);

  pthread_t tid;
  void *res;
  pthread_create(&tid, nullptr, sigHandlerThreadFun, nullptr);
  pthread_join(tid, &res);
  return 0;
}
```