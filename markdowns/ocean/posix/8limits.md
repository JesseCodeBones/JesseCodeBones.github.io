# limits
## API获取运行时限制
```C++
  auto result = pathconf(".", _PC_NAME_MAX);
  std::cout << result << std::endl;
```
## 命令行获取限制
```
(base) jesse@jesse-VirtualBox:~$ getconf -a | grep STACK
PTHREAD_STACK_MIN                  16384
_POSIX_THREAD_ATTR_STACKADDR       200809
_POSIX_THREAD_ATTR_STACKSIZE       200809
```