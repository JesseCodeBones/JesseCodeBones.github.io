# WASM 字节码规范
## 字节码规范解析
### 工具命令
查询详情： `wasm-objdump -x startblock.wasm`  
反汇编可执行文件： `wasm-objdump -d startblock.wasm`  
打印文件头部信息： `wasm-objdump -h startblock.wasm`  
通过wat生成wasm和source map: `wasm-opt debug.wat -o build/out.wasm -osm build/out.wasm.map -g -q`  
通过wasm 和 source map, 打印wat信息： `wasm-opt build/out.wasm -ism build/out.wasm.map -q -o - -S`

### WASM Binary Doc 解析
```
datasec = section (vec(code))
code = size:u32 code:func
func = vec(locals) e:expr
locals = n:u32 t:valtype
section = N:byte size:u32 cont:B
```
解释：
data section是有section 和vec(code) 组成
section的构成方式是1byte(type) _ LEB128(size n) _ n bytes
其中n bytes包含 vet(code)
vet的结构是 1byte(size n) _ n bytes (contents)
code的结构是 LEB128(size n) _ n bytes (contents)
func的结构是 vec(locals) _ expr这里展开就是1bytes length + locals + exprs

### 实例1
```
00 61 73 6D 01 00 00 00 01 84 80 80 80 00 01 60 
00 00 03 82 80 80 80 00 01 00 08 81 80 80 80 00 
00 0A 89 80 80 80 00 01 83 80 80 80 00 00 00 0B 
```
* 头部 `00 61 73 6D 01 00 00 00`
  - `00 61 73 6D` magic number
  - `01 00 00 00` version 1
* type section `01 84 80 80 80 00 01 60 00 00 `
  - 01 type section
  - `84 80 80 80 00` LEB128, length 4
  - `01 60 00 00` 01: 1 element, 60: function prototype, 00: 0 parameter, 00: 0 return value
* function section `03 82 80 80 80 00 01 00`
   - 03: function section
   - `82 80 80 80 00` length 2
   - `01 00` 01: 1 elements, 00 type index [0]
* start section `08 81 80 80 80 00 00`
  - 08 start section
  - `81 80 80 80 00` length 1
  - `00` function index 0
* code section `0A 89 80 80 80 00 01 83 80 80 80 00 00 00 0B`  
  - 0A code section 
  - `89 80 80 80 00` length 9
  - `01 83 80 80 80 00 00 00 0B` 01: size 1, `83 80 80 80 00`: size, `00`, local zero, `00 0B` => {unreachable; end;}

### 大小端转换
小端往小端或者大端往大端拷贝是可以直接用memcopy：  
`memcpy(out_value, state_.data + state_.offset, sizeof(T));`  
不同端之间的拷贝是要多加一步：   
```C 
memcpy(tmp, state_.data + state_.offset, sizeof(tmp));
  SwapBytesSized(tmp, sizeof(tmp));
  memcpy(out_value, tmp, sizeof(T));
```
```C++
inline void SwapBytesSized(void* addr, size_t size) {
  auto bytes = static_cast<uint8_t*>(addr);
  for (size_t i = 0; i < size / 2; i++) {
    std::swap(bytes[i], bytes[size - 1 - i]);
  }
}
```

## 使用Bynaryen添加一个方法
```C++
BinaryenType ii[2] = {BinaryenTypeInt32(), BinaryenTypeInt32()};
  BinaryenType params = BinaryenTypeCreate(ii, 2);
  BinaryenType results = BinaryenTypeInt32();
  BinaryenModuleRef module = BinaryenModuleRead(buffer, fsize);

  BinaryenExpressionRef x = BinaryenLocalGet(module, 0, BinaryenTypeInt32()),
                        y = BinaryenLocalGet(module, 1, BinaryenTypeInt32());
  BinaryenExpressionRef add = BinaryenBinary(module, BinaryenAddInt32(), x, y);
  BinaryenExpressionRef callOperands2[] = {makeInt32(module, 13),
                                           makeInt32(module, 3)};
  BinaryenFunctionRef adder =
      BinaryenAddFunction(module, "adder", params, results, NULL, 0, add);
  
```

## Binaryen parser的主要逻辑
`SExpressionWasmBuilder::SExpressionWasmBuilder`

## Binaryen lit 测试
`python build/bin/binaryen-lit -vv test/lit/source-map-drop.wast`  
