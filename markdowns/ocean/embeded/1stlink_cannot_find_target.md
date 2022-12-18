# STLINK 找不到target错误
很可能是STLink中可以用的频率和默认值不同，特别是keil中，它不可以通过stlink发现可用频率  
可以使用stlink utility然后进行链接，选择jtag然后频率会根据芯片不同进行调整  
关于如果芯片上运行了非SWD模式的软件会关掉SWD模式的问题  
可以通过将BOOT0接到5v，然后重启重新连接，让芯片启动到其他硬件硬盘上去。  
