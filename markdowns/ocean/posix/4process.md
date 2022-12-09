# process
进程（process）是一个可执行程序（program）的实例。  

从内核角度看，进程由用户内存空间（user-space memory）和一系列内核数据结构组成，其
中用户内存空间包含了程序代码及代码所使用的变量，而内核数据结构则用于维护进程状态信息。
记录在内核数据结构中的信息包括许多与进程相关的标识号（IDs）、虚拟内存表、打开文件的描
述符表、信号传递及处理的有关信息、进程资源使用及限制、当前工作目录和大量的其他信息。  

## 获取当前pid
```
pid_t pid = getpid();
printf("%d\n", pid);
pid_t parentPid = getppid();
printf("%d\n", parentPid);
```

如果子进程的父进程终止，则子进程就会变成“孤儿”，init 进程随即将收养该进程，子
进程后续对 getppid()的调用将返回进程号 1  

## 进程的内存布局
 文本段包含了进程运行的程序机器语言指令。文本段具有只读属性，以防止进程通过错误指针意外修改自身指令。因为多个进程可同时运行同一程序，所以又将文本段设为可共享，这样，一份程序代码的拷贝可以映射到所有这些进程的虚拟地址空间中。  
 初始化数据段包含显式初始化的全局变量和静态变量。当程序加载到内存时，从可执
行文件中读取这些变量的值。  
未初始化数据段包含了未进行显式初始化的全局变量和静态变量。此段常被称为 BSS 段，将经过初始化的全局变量和静态变量与未经初始化的全局变量和静态变量分开存放，其主要原因在于程序在磁盘上存储时，没有必要为未经初始化的变量分配存储空间。相反，可执行文件只需记录未初始化数据段的位置及所需大小，直到运行时再由程序加载器来分配这一空间。  

通过size命令可以查看各个segment的大小  
```
size cmaketest 
text	   data	    bss	    dec	    hex filename
6893	    800	    280	   7973	   1f25	cmaketest
```

### 内存情况demo

```C
char data[65535]; // bss
int name[] = {1, 2, 3}; // data
int add(int a, int b) { // stack frame
  int c = a + b; // stack frame
  return c; // register
}

int main() {
  static int key = 123; // data
  static int buf[123]; // bss
  int *d = (int *)malloc(sizeof(int)); //heap
  *d = add(1, 2);
  printf("%d\n", *d);
  free(d);
}
```
### 虚拟内存布局

![image](https://user-images.githubusercontent.com/56120624/202970904-3ba47267-a08b-416f-9d00-c5d076e22469.png)

任一时刻，每个程序仅有部分页需要驻留在物理内存页帧中。这些页构成了所谓驻留集（resident set）。程序未使用的页拷贝保存在交换区（swap area）内—这是磁盘空间中的保留区域，作为计算机 RAM 的补充—仅在需要时才会载入物理内存。若进程欲访问的页面目前并未驻留在物理内存中，将会发生页面错误（page fault），内核即刻挂起进程的执行，同时从磁盘中将该页面载入内存。  

## 共享内存
内存共享常发生于如下两种场景。 –执行同一程序的多个进程，可共享一份（只读的）程序代码副本。当多个程序执 行相同的程序文件（或加载相同的共享库）时，会隐式地实现这一类型的共享。 –进程可以使用 shmget()和 mmap()系统调用显式地请求与其他进程共享内存区。 这么做是出于进程间通信的目的。  

## 命令行
查看莫个进程的命令行
```
/proc/2448$ cat ./cmdline 
/usr/share/code/code --type=zygote --enable-crashpad --enable-crashpad
/proc/2448$ cat ./environ
SHELL=/bin/bashSESSION_MANAGER=local/jesse-VirtualBox...
```
可以可使用`extern char **environ`访问environment
```
extern char ** environ;
int main() {
  char * result = getenv("LC_NUMERIC");
  char ** env = environ;
  for (env; *env; env++)
  {
     std::cout << *env << std::endl;
  }
  std::cout << result << std::endl;
}
```
设置environment  
`putenv(char *) // key=value`
`setenv(char * name, char * value)` 
`unsetenv(char * name)`  
setenv()函数和 unsetenv()函数均来自 BSD，不如 putenv()函数使用普遍。尽管起初的POSIX.1 标准和 SUSv2 并未定义这两个函数，但 SUSv3 已将其纳入规范。  

## setjmp & longjmp
C 语言的 goto 语句存在一个限制，即不能从当前函数跳转到另一函数。  

## /proc文件信息
|文 件 |描述（进程属性）|
|-|-|
|cmdline| 以\0 分隔的命令行参数|
|cwd |指向当前工作目录的符号链接|
|Environ| NAME=value 键值对环境列表，以\0 分隔|
|exe |指向正在执行文件的符号链接|
|fd |文件目录，包含了指向由进程打开文件的符号链接|
|maps / smaps |内存映射|
|mem |进程虚拟内存（在 I/O 操作之前必须调用 lseek()移至有效偏移量）|
|mounts |进程的安装点|
|root |指向根目录的符号链接|
|status |各种信息（比如，进程 ID、凭证、内存使用量、信|号）|
|task |为进程中的每个线程均包含一个子目录（始自 |Linux 2.6）|

## /proc 子目录信息

|目 录 |目录中文件表达的信息|
|-|-|
|/proc|各种系统信息|
|/proc/net|有关网络和套接字的状态信息|
|/proc/sys/fs|文件系统相关设置|
|/proc/sys/kernel|各种常规的内核设置|
|/proc/sys/net|网络和套接字的设置|
|/proc/sys/vm|内存管理设置|
|/proc/sysvipc|有关 System V IPC 对象的信息|


## 查看uname系统信息
```C++
utsname n{};
uname(&n);
std::cout << n.sysname << "-" << n.machine << std::endl;
```
## 可重入函数 （reentrant）
可重入函数就是线程安全函数
* 同时试图更新同一全局变量或数据类型, 如malloc free
* 将静态数据结构用于内部记账的函数也是不可重入的，比如printf()  
* 信号处理器函数和主程序都要更新由程序员自定义的全局性数据结构，那么对于主程序而言，这种信号处理器函数就是不可重入的。

## 信号定时器


```C++
static volatile sig_atomic_t gotAlarm = 0;

int main(int argc, char **argv) {
  struct itimerval itv;
  clock_t preClock;
  struct sigaction sa;
  sigemptyset(&sa.sa_mask);
  sa.sa_flags = 0;
  sa.sa_handler = [](int sig) { gotAlarm = 1; };
  if (sigaction(SIGALRM, &sa, NULL) == -1) {
    exit(1);
  }
  itv.it_interval.tv_sec = 1;
  itv.it_interval.tv_usec = 0;
  itv.it_value.tv_sec = 1;
  itv.it_value.tv_usec = 0;
  if (setitimer(ITIMER_REAL, &itv, 0) == -1) {
    exit(1);
  }
  clock_t prevClock = clock();
  for (;;) {

    if (gotAlarm) {
      clock_t now = clock();
      gotAlarm = 0;
      std::cout << "get alarm signal from signal handler\n"
                << now - preClock << std::endl;
      preClock = now;
    }
  }

  return 0;
}
```

`alarm()`  
调用 alarm()会覆盖对定时器的前一个设置。调用 alarm(0)可屏蔽现有定时器。

## 定时器使用场景
为阻塞操作设置超时  
当用户在一段时间内没有输入整行命令时，可能希望取消对终端的 read()操作  
## 休眠
低分辨率休眠 sleep  
考虑到可移植性，应避免将 sleep()和 alarm()以及 setitimer()混用  
因为有些sleep的实现是基于timer的，这样会冲掉之前设置的定时器。
## posix时钟
调用此 API 的程序必须以-lrt 选项进行编译，也就是librt  
clock_gettime() clock_getres() clock_settime()

## posix定时器
* 以系统调用 timer_create()创建一个新定时器，并定义其到期时对进程的通知方法。
* 以系统调用 timer_settime()来启动或停止一个定时器。
* 以系统调用 timer_delete()删除不再需要的定时器。  

it_value 指定了定时器首次到期的时间。如果 it_interval 的任一子字段非 0，那么这就是一个周期性定时器，在经历了由 it_value 指定的初次到期后，会按这些子字段指定的频率周期性到期。如果 it_interval 的下属字段均为 0，那么这个定时器将只到期一次  

## 定时器溢出
在捕获或接收相关信号之前，定时器到期多次。这可能是因为进程再次获得调度前的延时所致  
在接收信号后可以获取定时器溢出计数即在信号生成与接收之间发生的定时器到期额外次数  
如果上次收到信号后定时器发生了 3 次到期，那么溢出计数是 2  
`timer_getoverrun(timer_t timerid)`  

## 线程通知
SIGEV_THREAD 标志允许程序从一个独立的线程中调用函数来获取定时器到期通知。  

## 定时文件描述符 timerfd (linux)
因为可以使用 select()、poll()和 epoll()将这种文件描述符会同其他描述符一同进行监控  
`timerfd_create()` `timerfd_gettime()`  `timer_settime()`  
## timerfd
一旦以 timerfd_settime()启动了定时器，就可以从相应文件描述符中调用 read()来读取定时器的到期信息。出于这一目的，传给 read()的缓冲区必须足以容纳一个无符号 8 字节整型  
timerfd_settime()修改设置以后，或是最后一次执行 read()后，如果发生了一起到多起定时器到期事件，那么 read()会立即返回，且返回的缓冲区中包含了已发生的到期次数  
如果并无定时器到期，read()会一直阻塞直至产生下一个到期。也可以执行 fcntl()的 F_SETFL 操作（5.3 节）为文件描述符设置 O_NONBLOCK 标志，这时的读动作是非阻塞式的，且如果没有定时器到期，则返回错误，并将 errno 值置为 EAGAIN。  

实例，通过timerfd进行延时读写：  
```C++
#include <sys/timerfd.h>
#include <time.h>
int main(int argc, char **argv) {
  int timerfd = timerfd_create(CLOCK_REALTIME, 0);
  struct itimerspec its;
  its.it_value.tv_sec = 1;
  its.it_value.tv_nsec = 0;
  its.it_interval.tv_sec = 0; // will not rerun
  its.it_interval.tv_nsec = 0;
  timerfd_settime(timerfd, 0, &its, NULL);
  uint64_t numExp;
  size_t s = read(timerfd, &numExp, sizeof(numExp));
  std::cout << "run after 1 second\n";
  if (s != sizeof(uint64_t)) {
    exit(1);
  }
  return 0;
}
```



