# Memory Management
## 查看当前内存情况
```C++
void printMem(){
  FILE *file = fopen("/proc/self/smaps", "r");
  char buf[1024];
  int readN = 0;
  std::stringstream stm;
  while ((readN = fread(buf, 1024, 1, file)) > 0)
  {
    stm << buf;
  }
  std::cout << stm.str() << std::endl;
}
```
查看heap base  
`void * heapBasePtr = sbrk(0);`  
调用 sbrk()将 program break 在原有地址上增加从参数 increment 传入的大小。（在 Linux 中，sbrk()是在 brk()基础上实现的一个库函数。）用于声明 increment 的 intptr_t 类型属于整数数据类型。若调用成功，sbrk()返回前一个 program break 的地址。换言之，如果 program break 增加，那么返回值是指向这块新分配内存起始位置的指针。  

## malloc free
若无法分配内存（或许是因为已经抵达 program break 所能达到的地址上限），则 malloc()
返回 NULL，并设置 errno 以返回错误信息。虽然分配内存失败的可能性很小，但所有对 malloc()
以及后续提及的相关函数的调用都应对返回值进行错误检查。

函数 mallopt()能修改各项参数，以控制 malloc()所采用的算法。  
mallinfo()函数返回一个结构，其中包含由 malloc()分配内存的各种统计数据.  

## 在堆栈上分配内存：alloca()

