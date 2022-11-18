# Posix File System
### default file descripter
|文件描述符| 用 途 |POSIX 名称 |stdio 流|
|---|--------|--------------|---------|
|0 | 标准输入 | STDIN_FILENO | stdin |
|1 |标准输出| STDOUT_FILENO| stdout|
|2| 标准错误| STDERR_FILENO| stderr|

## File copy
```C++
TEST(FileTest, BasicAssertions) {
  int inputFd, outputFd, openFlags;
  mode_t filePerms;
  size_t numRead;
  uint8_t buffer[BUFFER_SIZE];
  inputFd = open("/home/jesse/workspace/posix_demo/build/tests/testFile", O_RDONLY);
  if (inputFd == -1) {
    fprintf(stderr, "%s", "cannot open file: testFile\n");
    exit(1);
  }

  openFlags = O_CREAT | O_WRONLY | O_TRUNC;
  filePerms = S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP | S_IROTH | S_IWOTH;
  // S_IRUSR：用户读权限
  // S_IWUSR：用户写权限
  // S_IRGRP：用户组读权限
  // S_IWGRP：用户组写权限
  // S_IROTH：其他组都权限
  // S_IWOTH：其他组写权限
  outputFd = open("/home/jesse/workspace/posix_demo/build/tests/testOutFile", openFlags, filePerms);
  // read and write
  while ((numRead = read(inputFd, buffer, BUFFER_SIZE)) > 0) {
    int writeNum = write(outputFd, buffer, numRead);
    if (writeNum != numRead) {
      fprintf(stderr, "%s", "write file failed\n");
      exit(1);
    }
  }
  if (numRead == -1) {
    fprintf(stderr, "%s", "read file failed\n");
    exit(1);
  }
  if (close(inputFd) == -1) {
    fprintf(stderr, "%s", "cannot close file: testFile\n");
    exit(1);
  }
  if (close(outputFd) == -1) {
    fprintf(stderr, "%s", "cannot close file: testFileOut\n");
    exit(1);
  }
}
```

## 访问模式
|访 问 模 式 |描 述|
|-|-|
|O_RDONLY| 以只读方式打开文件|
|O_WRONLY |以只写方式打开文件|
|O_RDWR| 以读写方式打开文件 |

打开或者新建一个读写文档，将文档清零，将文档设置成当前用户RW  
`open("filepath", O_RDWR | O_TRUNC | O_CREAT, S_IRUSR | S_IWUSR)`  
打开或者创建一个文档， 将输出内容放在文档后面
`open("filepath", O_RDWR | O_APPEND | O_CREAT, S_IRUSR | S_IWUSR)`  

标识符列表  

|标 志 |用 途 |统一 UNIX 规范版本|
|-|-|-|
|O_RDONLY|以只读方式打开|v3|
|O_WRONLY|以只写方式打开|v3|
|O_RDWR|以读写方式打开|v3|
|O_CLOEXEC| 设置 close-on-exec 标志（自 Linux 2.6.23 版本开始）| v4|
|O_CREAT|若文件不存在则创建之|v3|
|O_DIRECT|无缓冲的输入/输出||
|O_DIRECTORY|如果 pathname 不是目录，则失败| v4|
|O_EXCL|结合 O_CREAT 参数使用，专门用于创建文件 |v3|
|O_LARGEFILE|在 32 位系统中使用此标志打开大文件||
|O_NOATIME| 调用 read()时，不修改文件最近访问时间（自 Linux 2.6.8|
|版本开始）||
|O_NOCTTY| 不要让 pathname（所指向的终端设备）成为控制终端| v3||
|O_NOFOLLOW|对符号链接不予解引用|v4|
|O_TRUNC|截断已有文件，使其长度为零|v3 |
|O_APPEND|总在文件尾部追加数据|v3|
|O_ASYNC|当 I/O 操作可行时，产生信号（signal）通知进程|
|O_DSYNC|提供同步的 I/O 数据完整性（自 Linux 2.6.33 版本开始）| v3|
|O_NONBLOCK|以非阻塞方式打开|v3|
|O_SYNC|以同步方式写入文件|v3|

### 说明  
O_CREAT  
如果文件不存在，将创建一个新的空文件。即使文件以只读方式打开，此标志依然有效。  
O_DIRECT  
无系统缓冲的文件 I/O 操作。  
O_DIRECTORY  
如果 pathname 参数并非目录，将返回错误  
O_EXCL  
此标志与 O_CREAT 标志结合使用表明如果文件已经存在，则不会打开文件，且 open()
调用失败，并返回错误，错误号 errno 为 EEXIST。  
O_NOATIME（自 Linux 2.6.8 版本开始）  
在读文件时，不更新文件的最近访问时间（15.1 节中所描述的 st_atime 属性）。要使用
该标志，要么调用进程的有效用户 ID 必须与文件的拥有者相匹配，要么进程需要拥有特权
（CAP_FOWNER）。否则，open()调用失败，并返回错误，错误号 errno 为 EPERM。  
O_NONBLOCK  
以非阻塞方式打开文件  
O_SYNC  
以同步 I/O 方式打开文件  
O_TRUNC  
如果文件已经存在且为普通文件，那么将清空文件内容，将其长度置 0。  

## create
creat()系统调用根据 pathname 参数创建并打开一个文件，若文件已存在，则打开文件，并清
空文件内容，将其长度清 0。  


## lseek
SEEK_SET  
将文件偏移量设置为从文件头部起始点开始的 offset 个字节。  
SEEK_CUR  
相对于当前文件偏移量，将文件偏移量调整 offset 个字节1。  
SEEK_END  
将文件偏移量设置为起始于文件尾部的 offset 个字节。也就是说，offset 参数应该从文件
最后一个字节之后的下一个字节算起。  
```C++
  lseek(fd, 0, SEEK_SET); // start of file
  lseek(fd, 0, SEEK_END); // next byte after end of file
  lseek(fd, -1, SEEK_END); // latest byte of file
  lseek(fd, -10, SEEK_CUR); // 10 before current
  lseek(fd, 1000, SEEK_END); 
```

## 空洞文件
```
[root 03]# cat hole.txt
01234567ABCDEFGH

[root 03]# od -c hole.txt              
0000000   0   1   2   3   4   5   6   7  \0  \0  \0  \0  \0  \0  \0  \0
0000020  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0
0000040   A   B   C   D   E   F   G   H
```

## 检查创建的原子操作
结合 O_CREAT 和 O_EXCL 标志来一次性地调用 open()可以防止这种情况，因
为这确保了检查文件和创建文件的步骤属于一个单一的原子（即不可中断的）操作。  
需要将文件偏移量的移动与数据写操作纳入同一原子操作。在打开文
件时加入 O_APPEND 标志就可以保证这一点  
有些文件系统（例如 NFS）不支持 O_APPEND 标志。在这种情况下，内核会选择
按如上代码所示的方式，施之以非原子操作的调用序列，从而可能导致上述的文件脏写
入问题。
## fcntl 
fcntl()的用途之一是针对一个打开的文件，获取或修改其访问模式和状态标志  
```
unsigned int flags = fcntl(inputFd, F_GETFL);
if (flags & O_SYNC)
{
  printf("no sync");
}
```
还可修改fd的描述符

## fd与文件的关系
一个文件可以被多个fd打开
要理解具体情况如何，需要查看由内核维护的 3 个数据结构。  
* 进程级的文件描述符表。
* 系统级的打开文件表。
* 文件系统的 i-node 表。

每个文件系统都会为驻留其上的所有文件建立一个 i-node 表。具体如下。  
* 文件类型（例如，常规文件、套接字或 FIFO）和访问权限。
* 一个指针，指向该文件所持有的锁的列表。
* 文件的各种属性，包括文件大小以及与不同类型操作相关的时间戳。  
访问一个文件时，会在内存中为 i-node 创建一
个副本，其中记录了引用该 i-node 的打开文件句柄数量以及该 i-node 所在设备的主、从设
备号，还包括一些打开文件时与文件相关的临时属性，例如：文件锁。

![image](https://user-images.githubusercontent.com/56120624/202678508-7728bc9a-5432-4348-8ce3-8173ccb6fd50.png)


