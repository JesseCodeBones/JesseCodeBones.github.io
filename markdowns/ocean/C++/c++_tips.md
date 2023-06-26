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

### 不定长参数列表
```C++
void myAdd(int num, ...) {
  va_list vaList;
  uint32_t sum = 0;
  int i;
  va_start(vaList, num);
  for (i = 0; i < num; i++) {
    sum += va_arg(vaList, int);
  }
  va_end(vaList);
  printf("sum=%d\n", sum);
}

int main() {
  myAdd(20, 1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1);
}
```

### 完整读取文件内容到char*
```C++
  std::ifstream input("/home/jesse/workspace/cmake_exec_test/CMakeLists.txt");
  if (!input) {
    std::cerr << "cannot open file\n";
    return 1;
  }
  input.seekg(0, std::ios::end);
  uint32_t size{static_cast<uint32_t>(input.tellg()) + 1U};
  input.seekg(0, std::ios::beg);
  std::unique_ptr<char[]> content(new char[size]{});
  input.read(content.get(), size);
  input.close();
  printf("%s", content.get());
```
