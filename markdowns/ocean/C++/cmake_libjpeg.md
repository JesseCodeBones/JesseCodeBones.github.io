# include libjpeg with cmake
### 参考文档
https://cmake.org/cmake/help/latest/module/FetchContent.html
### 引入github项目
```
include(FetchContent)
FetchContent_Declare(
  libjpeg
  GIT_REPOSITORY https://github.com/libjpeg-turbo/libjpeg-turbo.git
  GIT_TAG 4f7a8afbb7839a82dd357b5128cfb04dac6a5986
)
```
download下来之后会放在：
`build/_deps/`
### 编译下来的项目
`FetchContent_MakeAvailable(libjpeg)`
### 链接编译好的lib到项目上去
`target_link_libraries(libjpeg-demo PUBLIC jpeg-static)`
### 将header加入到当前target上面
`target_include_directories(libjpeg-demo PUBLIC ${libjpeg_BINARY_DIR} ${libjpeg_SOURCE_DIR})`
