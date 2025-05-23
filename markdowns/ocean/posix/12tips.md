# Useful tips during develpment
## useful API from posix

### qsort() sort array
```C++
int testArray[] = {-10, -15, 3, 2, 8, 9};
  qsort(testArray, sizeof(testArray) / sizeof(testArray[0]), sizeof(int),
        [](const void *a, const void *b) { return *(int *)a - *(int *)b; });
  for (int i = 0; i < 6; ++i) {
    std::cout << testArray[i] << std::endl;
  }
```  

## C++ skill
### delegations
通过宏定义去除大量重复代码：  
```C++
enum class Type : uint8_t { head_type, body_type, feet_type };

int printhead() {
  std::cout << "find head section" << std::endl;
  return 0;
}
int printbody() {
  std::cout << "find body section" << std::endl;
  return 0;
}
int printfeet() {
  std::cout << "find feet section" << std::endl;
  return 0;
}

int main(int argc, char **argv) {
  Type type = Type::body_type;
  switch (type) {
#define DELEGATE(name) \
  case Type::name##_type: \
    return print##name();
#include "delegations.def"
  default:
    throw std::runtime_error("no found");
  }
}
```
delegations.def  
```C++
DELEGATE(head);
DELEGATE(body);
DELEGATE(feet);
#undef DELEGATE
```
### 动态定义指针引用
```C++
#ifdef __cplusplus
#define BINARYEN_REF(NAME)\
  namespace wasm {\
  class NAME;     \
  };              \
  typedef class wasm::NAME* Binaryen##NAME##Ref;
#else
#define BINARYEN_REF(NAME) typedef struct  Binaryen##NAME* Binaryen##NAME##Ref;
#endif
```

这个时候调用 `BINARYEN_REF(Expression);`就会生成一个`BinaryenExpressionRef` 的类型。  
其实质是`Express *`  

