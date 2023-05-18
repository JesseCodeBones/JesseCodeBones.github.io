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

### 防止隐式转换
```C++
template <typename U, typename T=uint64_t>
T doSomething(U a) {
  static_assert(std::is_same<T, U>::value);
  return a;
}

```