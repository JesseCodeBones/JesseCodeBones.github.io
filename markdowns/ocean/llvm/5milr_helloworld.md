# MLIR 初体验

### llvm.mlir 
```mlir
// 定义一个模块
module {
  // 定义一个函数，函数名为 add_example，接收两个 i32 类型的参数，返回一个 i32 类型的值
  llvm.func @add_example(%arg0: i32, %arg1: i32) -> i32 {
    // 执行加法操作
    %result = llvm.add %arg0, %arg1 : i32
    // 返回加法结果
    llvm.return %result : i32
  }
}
```

### 通过以下命令生成汇编
```bash
mlir-opt add_example.mlir -o add_example_opt.mlir
mlir-translate --mlir-to-llvmir add_example_opt.mlir -o add_example.ll
llc add_example.ll -o add_example.s
```

### 通过转换后mlir被转换为
```S
	.build_version macos, 15, 0
	.section	__TEXT,__text,regular,pure_instructions
	.globl	_add_example                    ; -- Begin function add_example
	.p2align	2
_add_example:                           ; @add_example
	.cfi_startproc
; %bb.0:
	add	w0, w0, w1
	ret
	.cfi_endproc
                                        ; -- End function
.subsections_via_symbols


```