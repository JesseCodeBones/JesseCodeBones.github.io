## 寄存器加立即数

## 格式
ADD <Wd|WSP>, <Wn|WSP>, #<imm>{, <shift>}

## 解释
<shift>	
Is the optional left shift to apply to the immediate, defaulting to LSL #0 and encoded in sh:

sh	<shift>
0	LSL #0
1	LSL #12