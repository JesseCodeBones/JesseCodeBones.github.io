# 多线程开发
## 在多线程中捕获异常
```C++
void threadFun(std::exception_ptr &e_ptr) {
  try {
    throw std::invalid_argument("wrong");
  } catch (const std::exception &e) {
    e_ptr = std::current_exception();
  }
}

int main() {
  std::exception_ptr e_ptr;
  std::thread t1{threadFun, std::ref(e_ptr)};
  t1.join();
  if (e_ptr) {
    std::cout << "error happened during sub thread~\n";
  }
}
```

## atomic操作
下面的代码是像将a增加2000次，但是实际上是不到的 
```C++
int main() {
  int a;
  auto f = [&a]() { // increase 1000 times in each thread
    for (size_t i = 0; i < 1000; i++) {
      ++a;
      std::this_thread::sleep_for(std::chrono::milliseconds(1));
    }
  };
  std::thread t1{f}, t2{f};
  t1.join();
  t2.join();
  std::cout << a << std::endl;
}
```
需要对int a进行原子操作  

```C++
int main() {
  std::atomic_int a(0);
  auto f = [&a]() { // increase 1000 times in each thread
    for (size_t i = 0; i < 1000; i++) {
      a.fetch_add(1);
      std::this_thread::sleep_for(std::chrono::milliseconds(1));
    }
  };
  std::thread t1{f}, t2{f};
  t1.join();
  t2.join();
  std::cout << a << std::endl;
}
```

