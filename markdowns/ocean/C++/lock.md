# std locker

## locker的分类
1. lock_guard
2. unique_lock
3. shared_lock
区别：lock_guard创建即上锁，不能手动干预，要等到函数执行结束自动释放。
unique_lock可以手动上锁和解锁，属于写锁
shared_lock是读写所，只读的话不上锁，一旦上了写锁，所有读写锁都会被上  

## 一个基本锁的实现
```C++
std::mutex m1;
std::mutex m2;
void process() {
  std::unique_lock<std::mutex> lock1(m1, std::defer_lock_t());
  std::unique_lock<std::mutex> lock2(m2, std::defer_lock_t());
  lock(lock1, lock2);
  std::cout << "doing something~!\n";
}
int main() {
  std::vector<std::thread> threads(3);
  for (auto &t : threads) {
    t = std::thread(process);
  }
  for (auto &t : threads) {
    t.join();
  }
  return 0;
}
```

## 多线程中之调用一次的控制
```C++
std::once_flag gOnceFlag;

void initOnce() { std::cout << "only print once \n"; }

void process() {
  std::call_once(gOnceFlag, initOnce);
  std::cout << "doing something~!\n";
}
int main() {
  std::vector<std::thread> threads(3);
  for (auto &t : threads) {
    t = std::thread(process);
  }
  for (auto &t : threads) {
    t.join();
  }
  return 0;
}
```
## init lock
```C++
void initFun(){
  std::cout << "init finished" << std::endl;
}

std::atomic<bool> initialized(false);
std::mutex mut;
void fun(){
  if(!initialized) { // not initialized
    std::unique_lock<std::mutex> lock(mut);
    if (!initialized) // not other thread finish the initialization during aquire locker
    {
      initFun();
      initialized = true;
    }
  }
  std::unique_lock<std::mutex> lock(mut);
  std::cout << "OK" << std::endl;
}

int main() {
  std::vector<std::thread> threads;
  for (size_t i = 0; i < 10; i++)
  {
    threads.push_back(std::thread{fun});
  }
  for (auto& thread : threads)
  {
    thread.join();
  }
}
```