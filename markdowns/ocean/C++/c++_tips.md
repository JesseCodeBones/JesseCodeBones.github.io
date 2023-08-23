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
  void printMeBest(T&& me){ // 函数定义层面的完美转发, 涉及到引用折叠
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

### 让方法只接收4字节参数
```C++
template<typename T>
void only4BytesParam(T&& value) {
  static_assert(sizeof(T) == 4, "only 4 byte param");
  printf("hello=%d\n", static_cast<uint32_t>(value));
}

int main() {
  uint32_t v1 = 12U;
  only4BytesParam(v1);
  int32_t v2 = -1;
  only4BytesParam(v2);
  only4BytesParam(0xffffffff);
  uint64_t hello = 0x1;
  //only4BytesParam(hello); compile error
}
```
### 绕过内存对齐，直接操作内存
```C++
std::array<uint8_t, sizeof(void *) + sizeof(uint32_t) + sizeof(void *)> arr;
uint32_t a = 42;
uint32_t *ptr = &a;
void *targetPtr = &arr;
std::memcpy(&arr[0], &ptr, sizeof(void *));//写在0位置
std::memcpy(&arr[sizeof(void *)], &a, sizeof(uint32_t));//写在后面
std::memcpy(&arr[sizeof(void *) + sizeof(uint32_t)], &targetPtr,
            sizeof(void *));
```

### C++一个进度打印程序
```C++
int main() {
  std::cout << "Process:\n";
  uint8_t percentage = 0;
  while (percentage <= 100) {
    uint8_t show = std::ceil(percentage/5);
    std::array<char, 21> printArray;
    std::fill(printArray.begin(), printArray.end(), ' ');
    printArray.at(20) = '\0';
    for (int i = 0; i <= show; ++i) {
      printArray.at(i) = '=';
    }
    std::cout << printArray.data() << std::to_string(percentage) <<"%\r";
    std::cout.flush();
    std::this_thread::sleep_for(std::chrono::milliseconds(10));
    percentage++;
  }
  std::cout << "\n";
  std::cout << "run finished\n";
  return 0;
}
```


### 右值引用可以传递给左值const引用声明的函数
```C++
void printInt(const int & param) {
  std::cout << param << std::endl;
}

int main() {
  printInt(getOne());
  return 0;
}
```