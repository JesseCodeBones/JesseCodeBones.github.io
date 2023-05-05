# ARM64 Assembly
## 代码
```
.globl main
main:
    sub sp, sp, #32 // 申请32字节栈空间
    stp X29, X30, [sp, #16] // X29(FP 栈底) X30(LR 返回保存起来)从sp偏移16的位置开始往上存，存储完结果是[X30][X29][0][0][SP]
    
    // 函数体
    add X29, sp, #16 // 设置新的栈底，因为printf是通过 X29(FP)来确定栈底来取参数的
    mov w8, wzr // printf 函数就可以将返回值放到 w8 寄存器中
    stur wzr, [X29, #-4] // 将栈底下面4个字节置空 [X30][X29]X29[0][]SP
    adrp x0, .L.str
    add x0, x0, :lo12:.L.str // 获取字符串基地址
    str w8, [sp, #8]
    bl printf
    ldr w8, [sp, #8]

    // 函数体结束
    mov w0, w8  // return 0
    ldp x29, x30, [sp, #16] // 将FP, LR恢复回来
    add sp, sp, #32 // 施放申请的栈空间
    ret // return

.L.str:
    .asciz  "Hello World!\n"
    .size   .L.str, 14
```  
## 运行
### compile
`aarch64-linux-gnu-gcc -o ./build/hello 1helloworld.S`  
### run
`qemu-aarch64 -L /usr/aarch64-linux-gnu ./build/hello`  

