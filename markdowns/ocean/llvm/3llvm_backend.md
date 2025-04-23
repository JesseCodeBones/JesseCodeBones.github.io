## LLVM backend
### TableGen 指令选择

llvm-project/llvm/lib/Target/AMDGPU/SOPInstructions.td 为例子  

```
  def S_NOT_B64 : SOP1_64 <"s_not_b64",
    [(set i64:$sdst, (UniformUnaryFrag<not> i64:$src0))]
  >;
```

S_NOT_B64
调用具有64位整数类型参数的not节点，并返回64位整型的结果。


以webassembly为例子
defm ADD : BinaryInt<add, "add ", 0x6a, 0x7c>;  

WebAssemblyDAGToDAGISel::Select(SDNode *Node)  

指令选择
