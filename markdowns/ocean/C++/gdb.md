# GDB 使用技巧
### dump memory
```bash
dump memory /home/jesse/workspace/ExportSymbol/build/memory.data  0x555555554000 0x555555559000
```

### 查看程序方法汇编
```bash
disassemble _start
```

### GDB获取全局变量的地址
```bash
print &p
```

### hex格式打印莫个地址的值
```bash
print/x *0x555555558010
```
