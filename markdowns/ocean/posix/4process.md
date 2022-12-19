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

## 多进程
### fork
fork的一般template  
```C++
pid_t childPid;
  switch (childPid = fork()) {
  case -1:
    exit(1);
  case 0:
    std::cout << "handle from child process \n";
  default:
    std::cout << "handle from parent process \n";
  }
```
如果子进程更新了文件偏移量，那么这种改变也会影响到父进程中相应的描述符  
如果不需要这种对文件描述符的共享方式，那么在设计应用程序时，应于 fork()调用后注意两点：其一，令父、子进程使用不同的文件描述符；其二，各自立即关闭不再使用的描述符（亦即那些经由其他进程使用的描述符）。  

### 调用函数而不改变进程的内存需求量
对主进程保护的调用方式  
```C++
int func() {
  std::cout << "error run \n";
  malloc(sizeof(int)); // 存在内存泄露
  return 1;
}

int main(int argc, char **argv) {
  pid_t childPid;
  int status;
  switch (childPid = fork()) {
  case -1:
    exit(1);
  case 0:
    std::cout << "handle from child process \n";
    func();
    exit(0); // not 0, or 0 cannot pass other value
  default:
    wait(&status);
    std::cout << "handle from parent process \n";
    std::cout << status << std::endl;
  }
  return 0;
}
```  

### 父子进程通过信号通信
`kill(getppid(), SYNC_SIG);` // 向父进程发送信号  

`sigsuspend()` // 父进程进行等待

## 程序退出
程序退出的细节：  
* 关闭所有打开文件描述符、目录流、信息目录描述符以及（字符集）转换描述符（见 iconv_open(3)手册页）
* 作为文件描述符关闭的后果之一，将释放该进程所持有的任何文件锁
* 分离（detach）任何已连接的 System V 共享内存段, 且对应于各段的 shm_nattch 计数器值将减一
* 进程为每个 System V 信号量所设置的 semadj 值将会被加到信号量值中
* 如果该进程是一个管理终端（terminal）的管理进程那么系统会向该终端前台进程组中的每个进程发送 SIGHUP 信号
* 将关闭该进程打开的任何 POSIX 有名信号量，类似于调用 sem_close()
* 将关闭该进程打开的任何 POSIX 消息队列，类似于调用 mq_close()
* 作为进程退出的后果之一，如果某进程组成为孤儿，且该组中存在任何已停止进程则组中所有进程都将收到 SIGHUP 信号，随之为 SIGCONT 信号
* 移除该进程通过 mlock()或 mlockall()（50.2 节）所建立的任何内存锁
* 取消该进程调用 mmap()所创建的任何内存映射（mapping）

### atexit()注册exit清理方法
glibc有一个非标准注册方法`on_exit()`

### 进程退出的缓冲区问题
stdio是以\n字符刷新缓冲区  
file缓冲区是以buffer来刷新缓存
需要在创建子进程时考虑其IO的缓存问题。  

## 监控子进程
### wait() waitpid() waitpid()
子进程因收到信号 SIGCONT 而恢复，并以 WCONTINUED 标志调用 waitpid()  
waitpid()可以通过type来调整wait逻辑  
wait3()和 wait4()执行与 waitpid()类似的工作，区别在于wait3()和 wait4()在参数 rusage 所指向的结构中返回终止子进程的资源使用情况。其中包括进程使用的 CPU 时间总量以及内存管理的统计数据。  
pid_t wait3(int *wstatus, int options, struct rusage *rusage);  
顺便提一嘴C语言的通用设计，rusage是调用这提前分配的内存，它负责初始化和销毁，而这个方法只是负责将该填的数据填入结构体。  

### 孤儿进程与僵尸进程
换言之，某一子进程的父进程终止后，对 getppid()的调用将返回 1  
在父进程执行 wait()之前，其子进程就已经终止，这将会发生什么？  
子进程已经结束，系统仍然允许其父进程在之后的某一时刻去执行 wait()，以确定该子进程是如何终止的。内核通过将子进程转为僵尸进程（zombie）来处理这种情况。释放子进程所把持的大部分资源，以便供其他进程重新使用。该进程所唯一保留的是内核进程表中的一条记录，其中包含了子进程 ID、终止状态、资源使用数据（36.1 节）等信息。  
另一方面，如果父进程未执行 wait()随即退出，那么 init 进程将接管子进程并自动调用 wait()从而从系统中移除僵尸进程  

如果父进程创建了某一子进程，但并未执行 wait()，那么在内核的进程表中将为该子进程永久保留一条记录。如果存在大量此类僵尸进程，它们势必将填满内核进程表，从而阻碍新进程的创建。既然无法用信号杀死僵尸进程，那么从系统中将其移除的唯一方法就是杀掉它们的父进程（或等待其父进程终止）
，此时 init 进程将接管和等待这些僵尸进程，从而从系统中将它们清理掉  

在设计长生命周期的父进程（例如：会创建众多子进程的网络服务器和 Shell）时，这些语义具有重要意义。换句话说，在此类应用中，父进程应执行 wait()方法，以确保系统总是能够清理那些死去的子进程，避免使其成为长寿僵尸。如 26.3.1 节所述，父进程在处理 SIGCHLD 信号时，对 wait()的调用既可同步，也可异步。  

父进程避免僵尸子进程的积累的方法：  
* 父进程调用不带 WNOHANG 标志的 wait()，或 waitpid()方法，此时如果尚无已经终止的子进程，那么调用将会阻塞。  
* 父进程周期性地调用带有 WNOHANG 标志的 waitpid()，执行针对已终止子进程的非阻塞式检查（轮询）。
* 推荐使用方法：为 SIGCHLD 建立信号处理程序  
无论一个子进程于何时终止，系统都会向其父进程发送 SIGCHLD 信号。对该信号的默认处理是将其忽略， 不过也可以安装信号处理程序来捕获它。在处理程序中，可以使用 wait() 来收拾僵尸进程。  
由于信号处理函数存在信号阻塞的问题， 在 SIGCHLD 处理程序内部循环以 WNOHANG 标志来调用 waitpid()， 直到再无其他终止的子进程需要处理为止  
```C++
while (waitpid(-1, NULL, WNOHANG)) {
    continue;
  }
```
一个简单的SIGCHLD信号处理函数  
```C++
static void sigChildHandler(int sig) {
  int status, savedErrorNo;
  pid_t childPid;
  savedErrorNo = errno;
  std::cout << "handleing sigchld\n";
  while ((childPid = waitpid(-1, &status, WNOHANG)) > 0) {
    std::cout << "no hang for pid=" << childPid << ", with status=" << status
              << std::endl;
    numLiveChildren--;
  }
  errno = savedErrorNo;
}
int main(int argc, char **argv) {
  struct sigaction sa;
  sigset_t blockMask, emptyMask;
  sa.sa_flags = 0;
  sa.sa_handler = sigChildHandler;
  sigemptyset(&sa.sa_mask);
  if (sigaction(SIGCHLD, &sa, NULL) == -1) {
    exit(1);
  }
  sigemptyset(&blockMask); // block  SIGCHLD before SIGSUSPEND
  sigaddset(&blockMask, SIGCHLD);
  if (sigprocmask(SIG_SETMASK, &blockMask, NULL) == -1) {
    exit(1);
  }

  for (int i = 0; i < 10; i++) {
    switch (fork()) {
    case -1:
      exit(1);
    case 0:
      //  child process
      sleep((rand() % 50) + 1);
      _exit(EXIT_SUCCESS);
    default:
      break;
    }
  }
  sigemptyset(&emptyMask);
  int sigCount = 0;
  while (numLiveChildren > 0) {
    if (sigsuspend(&emptyMask) == -1 && errno != EINTR) {
      exit(1);
    }
    std::cout << "waiting for sig" << std::endl;
  }

  return 0;
}
```

### SA_NOCLDWAIT
可在调用 sigaction()对 SIGCHLD 信号的处置进行设置时使用此标志。设置该标志的作用类似于将对 SIGCHLD 的处置置为 SIG_IGN 时的效果。  

## exec库函数
|函数|对程序文件的描述（-, p）|对参数的描述（v, l）|环境变量来源（e, -）|
|-|-|-|-|
|execve()|路径名|数组envp| 参数|
|execle()|路径名|列表envp| 参数|
|execlp()|文件名+PATH|列表调用者的|environ|
|execvp()|文件名+PATH|数组调用者的|environ|
|execv()|路径名|数组调用者的| environ|

|excel()|路径名|列表调用者的| environ|

`fexecve()` 通过打开文件描述付来运行程序  
内核为每个文件描述符提供了执行时关闭标志 执行 exec()时，会自动关闭该文件描述符如果调用 exec()失败，文件描述符则会保持打开  
```C++
int fd = open("CMakeCache.txt", O_RDWR);
  int flags = 0;
  flags = fcntl(fd, F_GETFD);
  std::cout << flags << std::endl;
  flags |= FD_CLOEXEC;
  std::cout << flags << std::endl;
  if (fcntl(fd, F_SETFD, flags) == -1) {
    exit(1);
  }
```



