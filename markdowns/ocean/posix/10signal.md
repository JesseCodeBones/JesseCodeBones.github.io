# signal handler
## /proc/PID/status
这些字段分别为 SigPnd（基于线程的等待信号）、ShdPnd（进程
级等待信号，始于 Linux 2.6）、SigBlk（阻塞信号）、SigIgn（忽略信号）和 SigCgt（捕获信
号）  

## 常用信号和默认行为
头文件<signal.h>  
* SIGABRT: terminate process and core dump
* SIGBUS: 总线错误
* SIGALRM: 调用alarm()或者setitimer()过期
* SIGCHLD: 子进程终止
* SIGHUP: 挂起
* SIGINT: 终止 ctrl+c
* SIGPWD: 电源故障信号
* SIGTERM: kill
* SIGTRAP: 实现断点调试功能

## 信号处理函数
`sighandler_t signal(int signum, sighandler_t handler)`
```C++
signal(SIGTSTP, [](int sig) {
  std::cout << "control z happened \n";
});
for (;;)
{
  /* code */
}

```  
`sigaction()`  
```C++
struct sigaction sa;
sigemptyset(&sa.sa_mask);
sa.sa_flags = SA_RESTART;
sa.sa_handler = [](int sig){
  std::cout << "control z happened \n";
};
if (sigaction(SIGTSTP, &sa, NULL) == -1) {
  exit(1);
}
for (;;)
{
  /* code */
}
```
signal()的行为在不同 UNIX 实现间存在差异（22.7
节），这也意味着对可移植性有所追求的程序绝不能使用此调用来建立信号处理器函数。故此，
sigaction()是建立信号处理器的首选 API（强力推荐）。
## 信号处理器
调用信号处理器程序，可能会随时打断主程序流程；内核代表进程来调用处理器程序，
当处理器返回时，主程序会在处理器打断的位置恢复执行。  
发送信号的函数：  
`kill`, `raise`, `pthread_kill`, `killpg`  
## 显示信号描述 strsignal()
## 设置信号描述函数 psignal()
## 信号集
多个信号可使用一个称之为信号集的数据结构来表示，其系统数据类型为 sigset_t  
`sigemptyset()`函数初始化一个未包含任何成员的信号集  
`sigfillset()`函数则初始化一个信号
集，使其包含所有信号（包括所有实时信号）  
 `sigaddset()`， `sigdelset()` 向信号集添加或删除单个信号  
 `sigismember()` 判断某个信号是否在信号集中  
 ## 信号掩码
 内核会为每个进程维护一个信号掩码，即一组信号，并将阻塞其针对该进程的传递，如果将遭阻塞的信号发送给某进程，那么对该信号的传递将延后，直至从进程信号掩码中移除该信号，从而解除阻塞为止。  

使用 `sigprocmask()`系统调用，随时可以显式向信号掩码中添加或移除信号
```C++
sigset_t blockSet, prevMask;
sigemptyset(&blockSet);
sigaddset(&blockSet, SIGTSTP);

if (sigprocmask(SIG_BLOCK, &blockSet, &prevMask) == -1) {
  exit(1);
}
```
这样进程就不再接受SIGTSTP信号了“ctrl+z”  

## 等待处理的信号segpending()
等待信号集只是一个掩码，仅表明一个信号是否发生，而未表明其发生的次数。换言之，如果同一信号在阻塞状态下产生多次，那么会将该信号记录在等待信号集中，并在稍后仅传递一次。  

## pause() 等待信号
```C++
struct sigaction sa;
sigemptyset(&sa.sa_mask);
sa.sa_flags = SA_RESTART;
sa.sa_handler = [](int sig) { std::cout << "control z happened \n"; };
if (sigaction(SIGTSTP, &sa, NULL) == -1) {
  exit(1);
}
for (;;) {
  int res = pause();
  std::cout << res << std::endl;
  std::cout << "hello world\n";
}
```

## 可重入函数 （reentrant）
## 异步信号安全函数
异步信号安全的函数是指当从信号处理器函数调用时，可以保证其实现是安全的。如
果某一函数是可重入的，又或者信号处理器函数无法将其中断时，就称该函数是异步信号安
全的。

longjmp()来退出信号处理器函数，那么信号掩码会发生什么情况呢?  
在 System V / Linux 一脉中，longjmp()不会将信号掩码恢复  
BSD 系统中setjmp()将信号掩码保存在其 env 参数中，而信号掩码的保存值由 longjmp()恢复  
BSD 的实现还提供另外两个拥有 System V 语义的函数：_setjmp()和_longjmp()。  

## 一场终止abort()
产生SIGABRT 信号来终止调用进程。  
无论阻塞或者忽略 SIGABRT 信号，abort()调用均不受影响。同时规定，除非进程捕获 SIGABRT 信号后信号处理器函数尚未返回，否则 abort()必须终止进程。

## 备用栈中信号处理函数sigaltstack()
当进程对栈的扩展试图突破其上限时，内核将为该进程产生 SIGSEGV 信号但是这个时候栈空间已经满了，不能处理栈溢出，所以需要做以下步骤：  
* 分配一块被称为“备选信号栈”的内存区域，作为信号处理器函数的栈帧。
* 调用 sigaltstack()，告之内核该备选信号栈的存在。
* 在创建信号处理器函数时指定 SA_ONSTACK 标志，亦即通知内核在备选栈上为处理器函数创建栈帧。

利用系统调用 sigaltstack()，既可以创建一个备选信号栈，也可以将已创建备选信号栈的相关信息返回。  

栈溢出通知并关闭程序：  
```C++
void overflowFun(int a) {
  int b[10000];
  overflowFun(a);
}
int main() {
  stack_t sigstack;
  struct sigaction sa;
  int j;
  sigstack.ss_sp = malloc(SIGSTKSZ);
  sigstack.ss_size = SIGSTKSZ;
  sigstack.ss_flags = 0;
  if (sigaltstack(&sigstack, NULL) == -1) {
    printf("create alt stack fail\n");
    exit(1);
  }
  sa.sa_handler = [](int sig) {
    int x;
    std::cout << "stack over flow happened - " << strsignal(sig) << std::endl;
    printf("Top of handler stack near     %10p\n", (void *)&x);
    fflush(NULL);
    _exit(EXIT_FAILURE); /* Can't return after SIGSEGV */
  };
  sigemptyset(&sa.sa_mask);
  sa.sa_flags = SA_ONSTACK;
  if (sigaction(SIGSEGV, &sa, NULL) == -1) {
    printf("register hanlder fail\n");
    exit(1);
  }
  overflowFun(1);
}
```
## 系统调用的中断与恢复
在阻塞式函数中存在信号调用，阻塞的调用信号会失败，但是这个失败的errno会设置成`EINTER`这个可以判断阻塞式系统调用是否是因为信号处理函数而中断  
可以从这种中断中进行回复，而其他错误，就只能做失败处理：  
```C++
while ((ready = select(nfds, &readfds, NULL,  NULL, pto)) == -1 && errno == EINTR)
        continue;                       /* Restart if interrupted by signal */
    if (ready == -1)                    /* Unexpected error */
        errExit("select");
```
更可取的做法是可以调用指定了 SA_RESTART 标志的 sigaction()来创建信号处理器函数，从而令内核代表进程自动重启系统调用，还无需处理系统调用可能返回的 EINTR 错误。  
```C 
  memset(&sa, 0, sizeof(sa));
  if (sigfillset(&sa.sa_mask))
    abort();
  sa.sa_handler = uv__signal_handler;
  sa.sa_flags = SA_RESTART; // auto restart system call after enterrupt
  if (oneshot)
    sa.sa_flags |= SA_RESETHAND;

  /* XXX save old action so we can restore it later on? */
  if (sigaction(signum, &sa, NULL))
    return UV__ERR(errno);
```
并非所有的系统调用都可以通过指定 SA_RESTART 来达到自动重启的目的。 

BSD4.2中，wait()和 waitpid()的调用，以及如下I/O 系统调用：read()、readv()、write()和阻塞的 ioctl()操作  
慢速设备包括终端（terminal）、管道（pipe）、FIFO 以及套接字（socket）  

Linux中
wait()、waitpid()、wait3()、wait4()和 waitid()。  
访问慢速设备时的 I/O 系统调用：read()、readv()、write()、writev()和 ioctl()。  
系统调用 open()  
用于socket的各种系统调用  
POSIX 消息队列进行 I/O 操作的系统调用mq_receive()、mq_timedreceive()、mq_send()和 mq_timedsend()  
用于设置文件锁的系统调用和库函数：flock()、fcntl()和 lockf()。  
Linux 特有系统调用 futex()的 FUTEX_WAIT 操作   
用于递减 POSIX 信号量的 sem_wait()和 sem_timedwait()函数  
用于同步 POSIX 线程的函数：pthread_mutex_lock()、pthread_mutex_trylock()、pthread_mutex_timedlock()、pthread_cond_wait()和 pthread_cond_timedwait()  

绝不会自动重启的系统调用：  
poll()、ppoll()、select()和 pselect()这些 I/O 多路复用调用。  
epoll_wait()和 epoll_pwait()系统调用  
io_getevents()系统调用  
操作 System V 消息队列和信号量的阻塞系统调用：semop()、semtimedop()、msgrcv()和 msgsnd()  
用于将进程挂起指定时间的系统调用和库函数：sleep()、nanosleep()和clock_nanosleep()  
特意设计用来等待某一信号到达的系统调用：pause()、sigsuspend()、sigtimedwait()和sigwaitinfo()  

Linux中即使没有信号处理函数的注册，一些信号也会造成阻塞系统调用的失败并且设置`errno = EINTR`  
例如epoll_pwait()、epoll_wait()  

## SIGKILL 和 SIGSTOP 
SIGKILL 信号的默认行为是终止一个进程，SIGSTOP 信号的默认行为是停止一个进程  
二者的默认行为均无法改变（sigaction）  
同样，也不能将这两个信号阻塞。这是一个深思熟虑的设计决定  

## 进程休眠
休眠状态分为两种：  
TASK_INTERRUPTIBLE 进程正在等待某一事件  
TASK_UNINTERRUPTIBLE 进程正在等待某些特定类型的事件  
TASK_KILLABLE：可以被杀死的TASK_UNINTERRUPTIBLE状态  
## 硬件信号
硬件异常可以产生 SIGBUS、SIGFPE、SIGILL和 SIGSEGV 信号  

## 当信号处理函数新型系统调用时（用户态内核态转换）
当多个解除了阻塞的信号正在等待传递时，如果在信号处理器函数执行期间发生了内核态和用户态之间的切换，那么将中断此处理器函数的执行，转而去调用第二个信号处理器函数（如此递进）  

