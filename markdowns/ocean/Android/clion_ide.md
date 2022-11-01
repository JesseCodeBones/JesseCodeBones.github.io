# develop aosp with CLION IDE
## documentation location
`aosp/build/soong/docs/clion.md`
## steps
1. add environment variables  
```
export SOONG_GEN_CMAKEFILES=1
export SOONG_GEN_CMAKEFILES_DEBUG=1
```
2. 重新编译lib  
`make jessetest`  
3. 在clion中引用build出来的cmake文件：  
`/aosp/out/development/ide/clion/external/ajessetest/jessetest-x86_64-android/CMakeLists.txt`  