## Cmake test coverage

### module file
[Link](./CodeCoverage.cmake)
### implementation
在test目录下的CmakeLists.txt中加入  
```cmake
if(CMAKE_COMPILER_IS_GNUCXX)
  LIST(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/CMakeModules)
  include(CodeCoverage)
  APPEND_COVERAGE_COMPILER_FLAGS()
  setup_target_for_coverage_gcovr_xml(NAME cov
                            EXECUTABLE my_executable
                            DEPENDENCIES my_executable
                            BASE_DIRECTORY "${PROJECT_SOURCE_DIR}/src")

  setup_target_for_coverage_gcovr_html(NAME cov_html 
                            EXECUTABLE my_executable
                            DEPENDENCIES my_executable
                            BASE_DIRECTORY "${PROJECT_SOURCE_DIR}/src")                              
endif()
```  

## Cmake warning check
### Module file
[Link](./AddWarningFlags.cmake)
### implementation
`add_warning_flag_to_target(my_executable)`

## Cmake Clang-tidy

### 创建.clang-tidy文件
[link](./.clang-tidy)

### cmake中配置需要检查的文件路径 并且打开enable-werror, enable-sanitize

```
if(ENABLE_CLANG_TIDY)
        set(CMAKE_CXX_CLANG_TIDY clang-tidy --config-file ${CMAKE_CURRENT_SOURCE_DIR}/.clang-tidy --header-filter=${CMAKE_CURRENT_SOURCE_DIR}/[src|test]*.*hpp)
if(ENABLE_WERROR)
    set(CMAKE_CXX_CLANG_TIDY ${CMAKE_CXX_CLANG_TIDY} -warnings-as-errors=*)
endif()
if(ENABLE_SANITIZE)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address,undefined,pointer-compare")
endif()
```
特别注意，这些设置要放在add_executable之前

## Clang-format check

`find src test  -regex ".*\\.\\(cpp\\|hpp\\|c\\|h\\)" | xargs clang-format -style=file --Werror --dry-run`  


