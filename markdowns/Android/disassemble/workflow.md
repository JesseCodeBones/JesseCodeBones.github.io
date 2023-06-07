### use Radare2 for disassemble
### analysis
`aaa`
### list function
`afl`
### 查找字节
`/b 03 00 00 00`
### Il2cpp dumper
https://github.com/Perfare/Il2CppDumper
### disassemble Il2cpp
`Il2CppDumper.exe C:\Users\QXZ2U2E\Downloads\test\Challenge1\lib\arm64-v8a\libil2cpp.so C:\Users\QXZ2U2E\Downloads\test\Challenge1\assets\bin\Data\Managed\Metadata\global-metadata.dat C:\Users\QXZ2U2E\Downloads\test\temp`

### found method 
`PlayerLife__Die`
`VarContainer$$getLives`
### disassemble decreaseLives
`pdf fcn.0087792c`
### modify sub to add
` 0x00877978      29050051       sub w9, w9, 1`
change value to `29050011`
repackage apk