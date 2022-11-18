# Posix system call
### process
为响应中断 0x80，内核会调用 system_call()例程（位于汇编文件 arch/i386/entry.S 中）来  
处理这次中断，具体如下。  
a） 在内核栈中保存寄存器值（参见 6.5 节）。  
b） 审核系统调用编号的有效性。  
c） 以系统调用编号对存放所有调用服务例程的列表（内核变量 sys_call_table）进行索引，  
发现并调用相应的系统调用服务例程。若系统调用服务例程带有参数，那么将首先检
查参数的有效性。例如，会检查地址指向用户空间的内存位置是否有效。随后，该服
务例程会执行必要的任务，这可能涉及对特定参数中指定地址处的值进行修改，以及
在用户内存和内核内存间传递数据（比如，在 I/O 操作中）
。最后，该服务例程会将结
果状态返回给 system_call()例程。  
d） 从内核栈中恢复各寄存器值，并将系统调用返回值置于栈中。  
e） 返回至外壳函数，同时将处理器切换回用户态。
### process image
![image](https://user-images.githubusercontent.com/56120624/202347276-0edcaa0b-63b8-47e2-be3f-bb717536c45d.png)

### 查看连接器位置
`ldd ./cmaketest`
```
ldd ./cmaketest 
	linux-vdso.so.1 (0x00007fff2afae000)
	libstdc++.so.6 => /lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f32a8c6a000)
	libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f32a8c4a000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f32a8a22000)
	libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f32a893b000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f32a8ead000)
``` 
