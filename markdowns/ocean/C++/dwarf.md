# DWARF

## Dwarf document
[link](https://dwarfstd.org/doc/Dwarf3.pdf)

## print dwarf info
`objdump --debugging targetfile.cpp.o`  
## machanism
.debug_info和.debug_abbrev一起共同生成debug信息，里面总体上是一个树装结构
.debug_line是地址和源文件的对应信息
.debug_str是debug_info中需要的所有的string表

## TAG DEMO
```
<2><49>: Abbrev Number: 3 (DW_TAG_variable)
    <4a>   DW_AT_name        : a
    <4c>   DW_AT_decl_file   : 1
    <4d>   DW_AT_decl_line   : 4
    <4e>   DW_AT_decl_column : 7
    <4f>   DW_AT_type        : <0x71>
    <53>   DW_AT_location    : 2 byte block: 91 6c 	(DW_OP_fbreg: -20)

```

## Debug line DEMO
```
 Line Number Statements:
  [0x0000005b]  Set column to 11
  [0x0000005d]  Extended opcode 2: set Address to 0x0
  [0x00000064]  Special opcode 7: advance Address by 0 to 0x0 and Line by 2 to 3
  [0x00000065]  Set column to 7
  [0x00000067]  Special opcode 230: advance Address by 16 to 0x10 and Line by 1 to 4
  [0x00000068]  Special opcode 104: advance Address by 7 to 0x17 and Line by 1 to 5
  [0x00000069]  Special opcode 104: advance Address by 7 to 0x1e and Line by 1 to 6
  [0x0000006a]  Set column to 10
  [0x0000006c]  Special opcode 160: advance Address by 11 to 0x29 and Line by 1 to 7
  [0x0000006d]  Set column to 1
  [0x0000006f]  Special opcode 48: advance Address by 3 to 0x2c and Line by 1 to 8
  [0x00000070]  Advance PC by 2 to 0x2e
  [0x00000072]  Extended opcode 1: End of Sequence
```