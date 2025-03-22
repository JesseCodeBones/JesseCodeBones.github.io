### build all
`cmake -S llvm -B build -G Ninja`  
`cd build`  
`ninja -j1`  


### build clang
`cmake -G Ninja -S clang -B build_clang -DLLVM_INCLUDE_TESTS=OFF -DCMAKE_BUILD_TYPE=Debug`   
`cd build_clang`  
`ninja -j1`  

### 带clang的整体编译
`cmake -S llvm -B build   -DCMAKE_BUILD_TYPE=Debug -DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra;lld" -DLLVM_INCLUDE_TESTS=OFF `
