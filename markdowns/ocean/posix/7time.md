# time and date
## 获取时间戳并打印当前时间
```C++
  auto t = time(NULL);
  std::cout << t  << std::endl;
  std::cout << ctime(&t) << std::endl;
```
```
1669182673
Wed Nov 23 13:51:13 2022
```
## 格式化时间
```C++
  char buf[1024];
  auto t = time(NULL);
  auto localTime = localtime(&t);
  auto success = strftime(buf, 1024, "%Y年", localTime); // 格式化函数
  std::cout << success << std::endl << buf << std::endl;
```
将格式化的时间反转成time结构体  
strptime()  

## 进程时间
分为用户时间和系统时间  
方法是`times()`
