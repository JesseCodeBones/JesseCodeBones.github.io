## add shift register

## 简介

寄存器rm位移后, 与rn相加， 放入rd  
ADD Wd, Wn, Wm{, shift #amount}

shift	解释：  
Is the optional shift type to be applied to the second source operand, defaulting to LSL and encoded in shift:

shift	shift
00	LSL
01	LSR
10	ASR
11	RESERVED

amount是imm6：  
amount  

For the 32-bit variant: is the shift amount, in the range 0 to 31, defaulting to 0 and encoded in the "imm6" field.

For the 64-bit variant: is the shift amount, in the range 0 to 63, defaulting to 0 and encoded in the "imm6" field.
