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
### C++应用折叠和完美转发
```C++

class JesseTest {
public:
  void printMe(uint32_t&& me){ // 调用层面通过std::forward可以实现完美转发
    printf("me=%d\n", me);
  }
  void test(uint32_t& me){}
  template<typename T>
  void printMeBest(T&& me){ // 函数定义层面的完美转发
    printf("best me=%d\n", static_cast<uint32_t>(me));
  }
  ~JesseTest() { printf("destruct\n"); }
};

int main() {
  JesseTest test;
  test.printMe(42);
  uint32_t me = 42;
  //test.printMe(me); 编译错误
  test.printMe(std::forward<uint32_t>(me)); // 自动转化为右值，而且不会触发copy
  test.printMe(std::move(me));
  //test.printMe(std::ref(me)); 编译错误
  test.test(std::ref(me));// ref处理单个引用
  //test.test(42);编译报错，单个引用并不接受右值引用
  test.printMeBest(me);
  test.printMeBest(std::ref(me));
  test.printMeBest(std::move(me));
  test.printMeBest(42);
}
```
