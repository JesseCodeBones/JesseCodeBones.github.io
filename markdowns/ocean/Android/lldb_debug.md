## using LLDB to debug android native lib
### push lldb-server
```
jesse@jesse-virtual-machine:~/aosp$ find -name lldb-server
./prebuilts/clang/host/linux-x86/clang-r450784d/runtimes_ndk_cxx/aarch64/lldb-server
./prebuilts/clang/host/linux-x86/clang-r450784d/runtimes_ndk_cxx/i386/lldb-server
./prebuilts/clang/host/linux-x86/clang-r450784d/runtimes_ndk_cxx/arm/lldb-server
./prebuilts/clang/host/linux-x86/clang-r450784d/runtimes_ndk_cxx/x86_64/lldb-server
```

### push and run lldb-server
1. `adb push ./prebuilts/clang/host/linux-x86/clang-r450784d/runtimes_ndk_cxx/x86_64/lldb-server /data/local/tmp/`  
2. `adb shell /data/local/tmp/lldb-server p --server --listen unix-abstract:///data/local/tmp/debug.sock` if there is no permission, please chmod the executable file first.  
### run lldb client
1. back to ubuntu environment
2. come to source file folder, my is `~/aosp/external/jessetest`  
3. run client:
```
platform list
platform select remote-android
platform status
platform connect unix-abstract-connect:///data/local/tmp/debug.sock
```
then we successfully connected to lldb-server with below log:  
```
(lldb) platform select remote-android
  Platform: remote-android
 Connected: no
(lldb) platform connect unix-abstract-connect:///data/local/tmp/debug.sock
  Platform: remote-android
    Triple: x86_64-unknown-linux-android
OS Version: 33 (5.15.41-android13-8-00205-gf1bf82c3dacd-ab8747247)
  Hostname: localhost
 Connected: yes
WorkingDir: /data/local/tmp
```
4. find and set executable  
in this step, we need to find unstripped version, that means this version contains debug info, otherwise, you cannot attach break point on it.  
my file location is: `/home/jesse/aosp/out/soong/.intermediates/external/ajessetest/jessetest/android_x86_64/unstripped/jessetest`  
then set the executable:  
`/home/jesse/aosp/out/soong/.intermediates/external/ajessetest/jessetest/android_x86_64/unstripped/jessetest`
5. setup breakpoint
`breakpoint set -f main.cpp -l 11`  
6. run the executable and entry the break point  
`r`  
the console will show:  
```
Process 2553 launched: '/home/jesse/aosp/out/soong/.intermediates/external/ajessetest/jessetest/android_x86_64/unstripped/jessetest' (x86_64)
warning: (x86_64) /home/jesse/.lldb/module_cache/remote-android/.cache/0BA64F6D-28B8-90EF-BF31-D3D283147280/libnetd_client.so No LZMA support found for reading .gnu_debugdata section
Process 2553 stopped
* thread #1, name = 'jessetest', stop reason = breakpoint 1.1
    frame #0: 0x0000555555555623 jessetest`main at main.cpp:11:9

```
7. use lldb command to debug and view the variable
```
b = breakpoint 设置断点
c = continue 继续运行
n = next 下一行
s = step 单步进入
f = finish 跳出

p [var] 打印变量值
var 显示所有局部变量
bt 打印调用栈
up 在调用栈中向上移一帧 older
down 在调用栈中下移一帧 newer

register 查看寄存器
memory 查看内存
```