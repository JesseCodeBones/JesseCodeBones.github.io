# C++ tips
### print memory condition
```C++
#include <fstream>
void printMemoryMap() {
  std::ifstream f("/proc/self/smaps");
  if (f.is_open()) {
    std::cout << f.rdbuf();
  }
}
```

### 编译器命令行加宏
`cc -D_MY_MICRA_=64 test.c`  
### attribute
[link](https://en.cppreference.com/w/cpp/language/attributes)  
`GUARDED_BY(handleCounterMutex)` 确保变量访问的时候被锁保护