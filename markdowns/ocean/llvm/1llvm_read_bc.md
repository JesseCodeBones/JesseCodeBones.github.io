## read bc file
```C++
#include <iostream>
#include "llvm/IR/LLVMContext.h"
#include "llvm/IR/Module.h"
#include "llvm/Bitcode/BitcodeReader.h"
#include "llvm/Support/MemoryBuffer.h"
#include "llvm/Support/SourceMgr.h"
int main(int, char**) {
    llvm::LLVMContext context;
    auto buffer = llvm::MemoryBuffer::getFile("/home/jesse/workspace/llvm_test/hello.bc");
    if (!buffer) {
        std::cerr << "Failed to open\n";
        return 1;
    }
    auto module = llvm::parseBitcodeFile(buffer->get()->getMemBufferRef(), context);
    if (!module) {
        std::cerr << "Failed to parse \n";
        return 1;
    }

    for (auto &func : *module.get()) {
        std::cout << "Function: " << func.getName().str() << "\n";
    }
}
```