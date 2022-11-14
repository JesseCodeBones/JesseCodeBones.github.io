# Cmake CrossPlatform build
### Motivition
build and test with aarch64 output on x86_64 environment.  
### Check all available tasks
`cmake --build . --target help`  
### run build with cross plat-form target
`cmake -B build -DCMAKE_C_COMPILER=aarch64-linux-gnu-gcc -DCMAKE_CXX_COMPILER=aarch64-linux-gnu-g++ -DCMAKE_CROSSCOMPILING_EMULATOR=qemu-aarch64`  
### build executable with makefile
`cd build`  
`make help` show the targets  
`make jitdemo` build executable  
### verify if the target is aarch64 version 
`readelf -h jitdemo`  
output:  
```
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Position-Independent Executable file)
  Machine:                           AArch64
  Version:                           0x1
  Entry point address:               0xb40
  Start of program headers:          64 (bytes into file)
  Start of section headers:          11984 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         9
  Size of section headers:           64 (bytes)
  Number of section headers:         28
  Section header string table index: 27
```
