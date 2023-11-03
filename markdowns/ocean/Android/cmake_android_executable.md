# Soong build system
Soong can generate CLion projects. This is intended for source code editing only. Build should still be done via make/m/mm(a)/mmm(a).
```
$ export SOONG_GEN_CMAKEFILES=1
$ export SOONG_GEN_CMAKEFILES_DEBUG=1
```

You can then trigger a full build:

`$ make -j64`

or build only the project you are interested in:

`$ make frameworks/native/service/libs/ui`

Projects are generated in the out directory. In the case of libui, the path would be:
`out/development/ide/clion/frameworks/native/libs/ui/libui-arm64-android/CMakeLists.txt`

Note: The generator creates one folder per targetname-architecture-os combination. In the case of libui you endup with two projects:

```
$ ls out/development/ide/clion/frameworks/native/libs/ui
libui-arm64-android libui-arm-android
```

### Edit multiple projects at once
create a root CMakeLists.txt for both platforms  
```
cmake_minimum_required(VERSION 3.6)
project(native)
add_subdirectory(services/surfaceflinger)
add_subdirectory(libs/ui/libui-arm64-android)
add_subdirectory(libs/gui/libgui-arm64-android)
```
