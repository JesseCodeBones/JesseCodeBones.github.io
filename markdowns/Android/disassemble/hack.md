### unpack apk
`apktool d <apk>`
### disassemble Il2cpp logic
using tool:  
`https://github.com/Perfare/Il2CppDumper`  
command:  
`Il2CppDumper.exe C:\Users\QXZ2U2E\Downloads\test\Challenge1\lib\arm64-v8a\libil2cpp.so C:\Users\QXZ2U2E\Downloads\test\Challenge1\assets\bin\Data\Managed\Metadata\global-metadata.dat C:\Users\QXZ2U2E\Downloads\test\temp`
### find correct function and its offset
```C
// Namespace: 
public static class VarContainer // TypeDefIndex: 5628
{
	// Fields
	private static int lifes; // 0x0

	// Methods

	// RVA: 0x877F04 Offset: 0x877F04 VA: 0x877F04
	public static int getLives() { }

	// RVA: 0x87792C Offset: 0x87792C VA: 0x87792C
	public static void decreaseLives() { }

	// RVA: 0x877EA0 Offset: 0x877EA0 VA: 0x877EA0
	public static void resetLives() { }

	// RVA: 0x877F5C Offset: 0x877F5C VA: 0x877F5C
	private static void .cctor() { }
}
```

### disassemble libil2cpp.so elf file with Radare2
`s 0x87792c`  
`pd 100`

find hack point `0x00877978`:  
```
0x00877970      085c40f9       ldr x8, [x0, 0xb8]          ; 0xda
│           0x00877974      090140b9       ldr w9, [x8]                ; 0xe2
│           0x00877978      29050051       sub w9, w9, 1
│           0x0087797c      090100b9       str w9, [x8]
│           0x00877980      f37b41a9       ldp x19, x30, [var_10h]
│           0x00877984      f40742f8       ldr x20, [sp], 0x20
└           0x00877988      c0035fd6       ret

```

### edit libil2cpp.so
` 0x00877978      29050051       sub w9, w9, 1`
change value to `29050011`  
that means `add w9, w9, 1`

### repackage apk with apktool
`./apktool.sh b ./Challenge1 -o Challenge1_hacked.apk`

### resign apk with jarsigner

`/home/jesse/Android/Sdk/build-tools/33.0.2/zipalign -f -v 4 Challenge1_hacked.apk Challenge1_signed.apk`

`/home/jesse/Android/Sdk/build-tools/33.0.2/apksigner sign --ks ~/Documents/keystore/jesse.jks --ks-key-alias jesse Challenge1_signed.apk`

### verify sign
`jarsigner -verify -verbose Challenge1_signed.apk`
