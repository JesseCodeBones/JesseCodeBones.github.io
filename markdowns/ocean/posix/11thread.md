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
