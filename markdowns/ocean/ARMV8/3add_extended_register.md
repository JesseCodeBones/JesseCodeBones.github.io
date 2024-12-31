## 扩展register 移位加法

语法格式：  
add Xd, Xn, Xm, extend, amount

将Xm 看为 extend 格式，分别为 有无符号的 byte, halfword, word, or doubleword  
按照该格式将数字切分好，扩展为 32 | 64 位， 如 0xffff 如果是byte 格式，先进行 0xffff & 0xff (byte 长度)，在扩展64位：0x00000000000000ff
然后对其进行amount 左移 位操作
然后与Xn进行加法，放入xd


例如  
armv8 add x0, x0, w1, uxtb #1
x0 = 0x1
w1 = 0xffff

uxtb是unsigned byte的意思
所以
w1 & 0xff = 0xff

移位操作
0xff << 1 = 0x1fe

再与x0相加
0x1fe + 0x1 = 0x1ff

放入x0。
结果位0x1ff = 511

