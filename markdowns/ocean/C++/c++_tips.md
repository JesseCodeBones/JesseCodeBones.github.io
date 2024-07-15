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

### 通过一个priority_queue边排序边添加元素
```C++
  auto myCompare = [](const int &a, const int &b) { return a < b; };
  std::priority_queue<int, std::vector<int>, decltype(myCompare)> myQueue(
      myCompare);
  myQueue.push(1);
  myQueue.push(3);
  myQueue.push(2);

  std::vector<int> target;

  while (!myQueue.empty()) {
    auto item = myQueue.top();
    target.push_back(item);
    myQueue.pop();
  }

  for (auto& item : target) {
    std::cout << item << std::endl;
  }
```

### C++ http server
```C++
#include <iostream>
#include <sstream>
#include <string>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <unistd.h>
#include "string.h"

const int PORT = 8889;

std::string generateResponse() {
   std::stringstream response;
   response << "HTTP/1.1 200 OK\r\n";
   response << "Content-Type: text/html; charset=UTF-8\r\n";
   response << "\r\n";
   response << "<html><body>";
   response << "<h1>Hello, World!</h1>";
   response << "</body></html>";
   return response.str();
}

int main() {
   // 创建socket
   int sockfd = socket(AF_INET, SOCK_STREAM, 0);
   if (sockfd == -1) {
       std::cerr << "Failed to create socket." << std::endl;
       return 1;
   }
   
   // 绑定socket到本地地址和端口
   sockaddr_in addr{};
   addr.sin_family = AF_INET;
   addr.sin_addr.s_addr = INADDR_ANY;
   addr.sin_port = htons(PORT);
   if (bind(sockfd, (struct sockaddr*)&addr, sizeof(addr)) == -1) {
       std::cerr << "Failed to bind socket to port " << PORT << std::endl;
       close(sockfd);
       return 1;
   }
   
   // 监听连接
   if (listen(sockfd, 10) == -1) {
       std::cerr << "Failed to listen on socket." << std::endl;
       close(sockfd);
       return 1;
   }
   
   std::cout << "Server is running on port " << PORT << std::endl;

   
   
   while (true) {
       sockaddr_in addr;
       socklen_t len;
       int clientfd = accept(sockfd, (sockaddr*)&addr, &len);
       printf("IP address is: %s\n", inet_ntoa(addr.sin_addr));
       printf("port is: %d\n", (int) ntohs(addr.sin_port));
       
       
       if (clientfd == -1) {
           std::cerr << "Failed to accept client connection." << std::endl;
           close(sockfd);
           return 1;
       }
       
       // 读取请求
       constexpr int BUFFER_SIZE = 1024;
       char buffer[BUFFER_SIZE];
       memset(buffer, 0, BUFFER_SIZE);
       if (read(clientfd, buffer, BUFFER_SIZE - 1) == -1) {
           std::cerr << "Failed to read request from client." << std::endl;
           close(clientfd);
           close(sockfd);
           return 1;
       }
       
       std::cout << "Received request:\n" << buffer << std::endl;
       
       // 生成响应
       std::string response = generateResponse();
       
       // 发送响应
       if (write(clientfd, response.c_str(), response.length()) == -1) {
           std::cerr << "Failed to send response to client." << std::endl;
           close(clientfd);
           close(sockfd);
           return 1;
       }
       
       // 关闭连接
       close(clientfd);
   }
   
   // 关闭socket
   close(sockfd);

   return 0;
}
```


### C++ GC demo (20)
```C++
#include <deque>
#include <queue>
#include <set>
#include <type_traits>
#include <vector>
 
void sweep_clear_gc();
 
struct object {
  virtual ~object() = default;
  void __visit() const;
  virtual void __visit_children() const {}
};
 
struct global : public object {
  global(object *obj) : m_obj(obj) {}
  object *m_obj;
};
 
global a{nullptr};
 
std::deque<std::set<object const *>> root{{&a}};
 
std::set<object const *> white{};
std::set<object const *> black{};
 
void object::__visit() const {
  white.erase(this);
  black.insert(this);
  __visit_children();
}
 
struct array : public object {
  std::vector<object *> elements{};
 
  virtual void __visit_children() const override {
    this->object::__visit_children();
    for (auto obj : elements) {
      obj->__visit();
    }
  }
};
 
void on_function_in() { root.push_back({}); }
void on_function_out() { root.pop_back(); }
 
template <class T>
concept Object = std::is_base_of_v<object, T>;
 
void push_stack_object(object const *obj) { root.back().insert(obj); }
template <Object T> T *create_stack_object() {
  auto obj = new T();
  push_stack_object(obj);
  return obj;
}
 
void mark() {
  for (auto &objs : root) {
    for (auto obj : objs) {
      obj->__visit();
    }
  }
}
void sweep() {
  for (auto obj : white) {
    delete obj;
  }
}
 
void sweep_clear_gc() {
  mark();
  sweep();
 
  std::swap(white, black);
}
 
// -----------------------------------------------------
// -----------------------------------------------------
// -----------------------------------------------------
 
void foo(array *arr) {
  on_function_in();
  // push_stack_object(arr);
 
  arr = create_stack_object<array>();
 
  on_function_out();
}
 
int main() {
  on_function_in();
 
  auto arr = create_stack_object<array>();
  foo(arr);
 
  on_function_out();
}
```

### C++ virtual studio static check
```cmake
function(EnableVisualStudioStaticCheck target)
    if((${CMAKE_GENERATOR} MATCHES "Visual Studio") AND ENABLE_CLANG_TIDY)
        set_target_properties(${target} PROPERTIES
            VS_GLOBAL_RunCodeAnalysis true
            VS_GLOBAL_EnableMicrosoftCodeAnalysis false
            VS_GLOBAL_EnableClangTidyCodeAnalysis true
        )
    endif()
endfunction()
```
```
if(ENABLE_CLANG_TIDY)
    EnableVisualStudioStaticCheck(${PROJECT_NAME})
    if(ENABLE_WERROR)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W2")
    endif()
endif()

```

### C++ memory barrier
1. 防止指令重排序

处理器和编译器可能会出于优化目的重排序指令。这种重排序在单线程环境中通常不会引起问题，但在多线程环境中可能会导致数据竞争和难以调试的错误。内存屏障可以防止这种重排序，确保指令按照程序员预期的顺序执行。
2. 确保内存可见性

在多处理器系统中，各个处理器可能会有自己的高速缓存，导致一个处理器上的修改对其他处理器不可见。内存屏障可以确保一个处理器对内存的更改在其他处理器上可见，从而保证数据的一致性。
3. 同步多线程操作

在多线程编程中，内存屏障用于确保线程之间的操作顺序。例如，生产者-消费者模型中，生产者线程需要确保在生产的数据对消费者线程可见之前，所有写操作都已经完成。内存屏障可以确保这种顺序。

```C++
#include <atomic>
#include <thread>
#include <iostream>

std::atomic<int> data[5];
std::atomic<bool> ready(false);

void producer() {
    data[0].store(1, std::memory_order_relaxed);
    data[1].store(2, std::memory_order_relaxed);
    data[2].store(3, std::memory_order_relaxed);
    data[3].store(4, std::memory_order_relaxed);
    data[4].store(5, std::memory_order_relaxed);
    
    std::atomic_thread_fence(std::memory_order_release);
    ready.store(true, std::memory_order_relaxed);
}

void consumer() {
    while (!ready.load(std::memory_order_relaxed)) {
        std::this_thread::yield();
    }
    
    std::atomic_thread_fence(std::memory_order_acquire);
    std::cout << "data[0]: " << data[0].load(std::memory_order_relaxed) << "\n";
    std::cout << "data[1]: " << data[1].load(std::memory_order_relaxed) << "\n";
    std::cout << "data[2]: " << data[2].load(std::memory_order_relaxed) << "\n";
    std::cout << "data[3]: " << data[3].load(std::memory_order_relaxed) << "\n";
    std::cout << "data[4]: " << data[4].load(std::memory_order_relaxed) << "\n";
}

int main() {
    std::thread t1(producer);
    std::thread t2(consumer);
    
    t1.join();
    t2.join();
    
    return 0;
}

```

### assert
assert only execute when debug mode is off 

### C++ vtable

``` C++
#include <cstddef>
#include <iostream>

class Base {
public:
  virtual void print() { std::cout << "Base class print function \n"; }
};

class Derived : public Base {
public:
  virtual void childPrint() { std::cout << "Derived class childPrint function \n"; } // in vtable, it is the second element
  void print() { std::cout << "Derived class print function \n"; } // it is the first element, so it is the same as Base class print function
};

int main() {
  Base *base = new Base();
  Derived *derived = new Derived();
  void (*printFunBase)() = NULL;
  void (*printFunDerived)() = NULL;
  void *vptrBase = *(void **)base;
  void *vptrDerived = *(void **)derived;
  std::cout << "Base vptr: " << vptrBase << std::endl;
  std::cout << "Derived vptr: " << vptrDerived << std::endl;

  std::cout << "Base vptr value: " << (void *)*((void **)vptrBase) << std::endl;
  std::cout << "Derived vptr value: " << (void *)*((void **)vptrDerived)
            << std::endl;
  void *baseFunPtr = (void *)*((void **)vptrBase);
  void *derivedFunPtr = (void *)*((void **)vptrDerived);
  auto * childPrintPtr = *((void **)vptrDerived + 1);
  std::cout << "Base vptr value of childPrintPtr: " << childPrintPtr << std::endl;
  return 0;
}

```
