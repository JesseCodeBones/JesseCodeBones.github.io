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
O_EXCL至关重要  
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

## dup & dup2
dup()调用复制一个打开的文件描述符 oldfd，并返回一个新描述符，二者都指向同一打开
的文件句柄。系统会保证新描述符一定是编号值最低的未用文件描述符。  
dup2()系统调用会为 oldfd 参数所指定的文件描述符创建副本，其编号由 newfd 参数指定。
如果由 newfd 参数所指定编号的文件描述符之前已经打开，那么 dup2()会首先将其关闭。  

## pread & pwrite
系统调用 pread()和 pwrite()完成与 read()和 write()相类似的工作，只是前两者会在 offset 参数
所指定的位置进行文件 I/O 操作，而非始于文件的当前偏移量处，且它们不会改变文件的当前
偏移量。  

## readv & writev

批量读写

```
  iovec buffers[2];
  char *data = "hello world";
  char *data2 = "I am jesse";
  iovec buf1{data, 12};
  iovec buf2{data2, 11};
  buffers[0] = buf1;
  buffers[1] = buf2;
  writev(1, buffers, 2);
  return 0;
```
原子性是 readv()的重要属性。换言之，从调用进程的角度来看，当调用 readv()时，内核
在 fd 所指代的文件与用户内存之间一次性地完成了数据转移。这意味着，假设即使有另一进
程（或线程）与其共享同一文件偏移量，且在调用 readv()的同时企图修改文件偏移量，readv()
所读取的数据仍将是连续的。  
调用 readv()成功将返回读取的字节数，若文件结束1
将返回 0。调用者必须对返回值进行
检查，以验证读取的字节数是否满足要求。  
writev()调用也属于原子操作，即所有数据将一次性地从用户内存传
输到 fd 指代的文件中。因此，在向普通文件写入数据时，writev()调用会把所有的请求数据连
续写入文件，而不会在其他进程（或线程）写操作的影响下1
分散地写入文件2
。  

## truncate()和 ftruncate()
若文件当前长度大于参数 length，调用将丢弃超出部分，若小于参数 length，调用将在文
件尾部添加一系列空字节或是一个文件空洞。  

## /dev/fd

对于每个进程，内核都提供有一个特殊的虚拟目录/dev/fd.  
/dev/fd 实际上是一个符号链接，链接到 Linux 所专有的/proc/self/fd 目录  

## 临时文件
```C++
  int fd;
  char templatestr[] = "/tmp/anythingXXXXXX"; // system will replace XXXXXX with real file name
  fd = mkstemp(templatestr);
  if (fd == -1)
    exit(1);
  printf("generated file name is %s\n", templatestr);
  unlink(templatestr);
```

tmpfile()函数会创建一个名称唯一的临时文件，并以读写方式将其打开。（打开该文件时
使用了 O_EXCL 标志，以防一个可能性极小的冲突，即另一个进程已经创建了一个同名文件。）  
tmpfile()函数执行成功，将返回一个文件流供 stdio 库函数使用。文件流关闭后将自动删
除临时文件。为达到这一目的，tmpfile()函数会在打开文件后，从内部立即调用 unlink()来删
除该文件名1
。  

```C++
  FILE *tmp = tmpfile();
  const char *jessetest = "hello world";
  fwrite(jessetest, 12, 1, tmp);
  fflush(tmp);
  fseek(tmp, 0, SEEK_SET); // seek to the beginning
  char result[12];
  int readCnt = fread(result, 12, 1, tmp);
  printf("%s\n", result);
  printf("%d\n", readCnt);
```

## 缓冲区时间
复制一个100MB的文件使用不同的buffer用时的不同
|总用时| 总 CPU 用时| 用户 CPU 用时 |系统 CPU 用时|
|-|-|-|-|
|1|107.43|107.32|8.20|99.12|
|2|54.16|53.89|4.13|49.76|
|4|31.72|30.96|2.30|28.66|
|8|15.59|14.34|1.08|13.26|
|16|7.50|7.14|0.51|6.63|
|32|3.76|3.68|0.26|3.41|
|64|2.19|2.04|0.13|1.91|
|128|2.16|1.59|0.11|1.48|
|256|2.06|1.75|0.10|1.65|
|512|2.06|1.03|0.05|0.98|
|1024|2.05|0.65|0.02|0.63|
|4096|2.05|0.38|0.01|0.38|
|16384|2.05|0.34|0.00|0.33|
|65536|2.06|0.32|0.00|0.32|

写入一个100MB的文件普遍比复制一个100MB的文件要用时短，原因是写入的时候只是写入到内核管理内存上去，并没有真正的通过磁盘写入磁盘，而复制却多了一步从磁盘上读取数据的过程，造成耗时增多。

## stdio缓冲
`setvbuf()`可以控制stdio的缓冲大小。  
_IONBF
不对 I/O 进行缓冲。每个 stdio 库函数将立即调用 write()或者 read()，并且忽略 buf 和 size
参数，可以分别指定两个参数为 NULL 和 0。stderr 默认属于这一类型，从而保证错误能立即
输出。  

_IOLBF
采用行缓冲 I/O。指代终端设备的流默认属于这一类型。对于输出流，在输出一个换行符
（除非缓冲区已经填满）前将缓冲数据。对于输入流，每次读取一行数据。 

_IOFBF
采用全缓冲 I/O。单次读、写数据（通过 read()或 write()系统调用）的大小与缓冲区相同。
指代磁盘的流默认采用此模式。

`fflush()`库函数，强制刷新缓冲区到内核缓冲区，但不一定到文件系统。关闭流时也会自动刷新缓冲区。

## 内核缓冲到磁盘
`fsync()`使缓冲数据和与打开文件描述符 fd 相关的所有元数据都刷新到磁盘上  

`fdatasync()`强制做一个系统级data sync  
fdatasync()可能会减少对磁盘操作的次数，由 fsync()调用请求的两次变为一次。例如，若
修改了文件数据，而文件大小不变，那么调用 fdatasync()只强制进行了数据更新。（前面已然
述及，针对 synchronized I/O data completion 状态，如果是诸如最近修改时间戳之类的元数据
属性发生了变化，那么是无需传递到磁盘的。）相比之下，fsync()调用会强制将元数据传递到
磁盘上。

`sync()`会将文件信息更新到磁盘上  
若内容发生变化的内核缓冲区在 30 秒内未经显式方式同步到磁盘上，则一条长期运行的
内核线程会确保将其刷新到磁盘上。这一做法是为了规避缓冲区与相关磁盘文件内容长期处
于不一致状态（以至于在系统崩溃时发生数据丢失）的问题。在 Linux 2.6 版本中，该任务由
pdflush 内核线程执行。

## 使用O_SYNC flag进行写入同步
调用 open()函数时如指定 O_SYNC 标志，则会使所有后续输出同步（synchronous）。
调用 open()后，每个 write()调用会自动将文件数据和元数据刷新到磁盘上（即，按照
Synchronized I/O file integrity completion 的要求执行写操作）。

## O_DSYNC O_RSYNC 标志
synchronized I/O data integrity completion  
synchronized I/O file integrity completion  
O_RSYNC 标志是与 O_SYNC 标志或 O_DSYNC 标志配合一起使用的。

如果打开文件时指定 O_RSYNC 和 O_SYNC 标志在执行读操作之前，像执行 O_SYNC 标志一样完成所有待处理的写操作  

## 向内核提意见 posix_fadvise()
POSIX_FADV_NORMAL 无特别建议  
POSIX_FADV_SEQUENTIAL 低偏移量到高偏移量顺序读取数据，预读窗口大小置为默认值的两倍。  
POSIX_FADV_RANDOM 随机顺序访问数据。在 Linux 中，该选项会禁用文件预读  
POSIX_FADV_WILLNEED 不久的将来访问指定的文件区域， 内核将由 offset 和 len 指定区域的文件数据预先填充到缓冲区高速缓存中  
POSIX_FADV_DONTNEED 进程预计在不久的将来将不会访问指定的文件区域  
POSIX_FADV_NOREUSE 预计会一次性地访问指定文件区域，不再复用。这等于提示内核对指定区域访问一次后即可释放页面。在 Linux 中，该操作目前不起作用。

## O_DIRECT 直接IO
绕过kernal高速缓存  
对齐、长度及偏移量必须是底层文件系统逻辑块大小的整数倍。（典型文件系统的逻辑块大小为 1024、2048 或 4096
字节。）  
* 用于传递数据的缓冲区，其内存边界必须对齐为块大小的整数倍。
* 数据传输的开始点，亦即文件和设备的偏移量，必须是块大小的整数倍。
* 待传递数据的长度必须是块大小的整数倍。

## std 和 posix混合使用
`int fileno(FILE* file) // get fd from std file`  
`FILE* fdopen(int fd)` 

## stat() lstat() fstat()

获取file的状态信息
```C++
struct stat s{};
stat("./Makefile", &s);
std::cout << 512 * s.st_blocks << std::endl;
```
占用BLK的大小

st_blksize 字段的命名多少有些令人费解。其所指并非底层文件系统的块大小，而是针对文件系统上文件进行 I/O 操作时的最优块大小（以字节为单位）。若 I/O 所采用的块大小小于该值，则被视为低效（参阅 13.1 节）。一般而言，st_blksize 的返回值为 4096。

utimensat()系统调用（内核自 2.6.22 版本开始支持）和 futimens()库函数（glibc 自版本 2.6开始支持）为设置对文件的上次访问和修改时间戳提供了扩展功能。
* 可按纳秒级精度设置时间戳。相对于提供微秒级精度的 utimes()，这是重大改进。
* 可独立设置某一时间戳（一次只设置其一）。如前所述，要使用旧编程接口去改变时间
戳之一，需要首先调用 stat()获取另一时间戳的值。然后再将获取值与打算变更的时间
戳一同指定。（若另一进程在这两步之间执行了更新时间戳的操作，将会导致竞争状态。）
* 可独立将任一时间戳置为当前时间。要使用旧编程接口将一个时间戳改为当前时间，Linux/UNIX 系统编程手册（上册）需要调用 stat()去获取那些保持不变的时间戳的设置情况，并调用 gettimeofday()以获得当前时间。

## 权限检查access
```C++
int exists = access("/etc", F_OK);
int writeable = access("/etc", W_OK);
std::cout << exists << std::endl; // 0
std::cout << writeable << std::endl; // -1
```

## ALC 访问控制列表
需要在为特定用户和组授权时进行更为精密的控制
可以在任意数量的用户和组之中，为单个用户或组指定文件权限。
每条 ACE 都由 3 部分组成。
* 标记类型：表示该记录作用于一个用户、组，还是其他类别的用户。
* 标记限定符（可选项）标识特定的用户或组（亦即，某个用户 ID 或组 ID）。
* 权限集合：本字段包含所授予的权限信息（读、写及执行）。

set ALC
`setfacl -m u:mail:r README.md`  
get ALC `getfacl README.md`  

## ALC APIs
acl_get_file()函数可用来获取（由 pathname 所标识）文件的 ACL 副本。  
acl_get_entry()会返回一句柄  
acl_get_tag_type()和 acl_set_tag_type()可获取修改ALC的值  
acl_create_entry()函数用来在某一现有 ACL 中新建一条记录  
acl_set_file()函数的作用与 acl_get_file()相反，将使用驻留于内存的 ACL 内容（由 acl 参数所指代）来更新磁盘上的 ACL。  

## 目录
目录和文件的区别
* 在其 i-node 条目中，会将目录标记为一种不同的文件类型（参见 14.4 节）。
* 目录是经特殊组织而成的文件。本质上说就是一个表格，包含文件名和 i-node 编号。

## 目录方法
link  
unlink  
rename  
symlink  
readlink  
mkdir  
rmdir  
remove 移除文件或者目录  
opendir  
readdir  
readdir_r  
nftw  遍历树  
getcwd  
chdir  
fchdir  
chroot  
realpath 解析路径名   
dirname, basename  解析路径名字符串  

## inotify API
监听目录或者文件事件  
`int inoti = inotify_init();`  
```C++
const char *const pathName = "/home/jesse";
int wd = inotify_add_watch(inoti, pathName, IN_ALL_EVENTS);
```  
`unsigned long long int numRead = read(inoti, buf, BUF_LEN);`  

实例代码[LINK](https://github.com/JesseCodeBones/tlpi-dist/blob/master/inotify/demo_inotify.c)  


  