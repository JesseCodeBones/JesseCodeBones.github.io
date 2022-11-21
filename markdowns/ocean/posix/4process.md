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

