## arm32
| instruction | description |
|---|---|
|ADC|带进位的32位数加法|
|ADD|32位数相加|
|AND|32位数的逻辑与|
|B|在32M空间内的相对跳转指令|
|BEQ|相等则跳转（Branch if EQual）|
|BNE|不相等则跳转（Branch if Not Equal）|
|BGE|大于或等于跳转（Branch if Greater than or Equa）|
|BGT|大于跳转（Branch if Greater Than）|
|BIC|32位数的逻辑位清零|
|BKPT|断点指令|
|BL|带链接的相对跳转指令|
|BLE|小于或等于跳转（Branch if Less than or Equal）|
|BLEQ|带链接等于跳转（Branch with Link if EQual）|
|BLLT|带链接小于跳转（Branch with Link if Less Than）|
|BLT|小于跳转（Branch if Less Than）|
|BLX|带链接的切换跳转|
|BX|切换跳转|
|CDP CDP2|协处理器数据处理操作|
|CLZ|零计数|
|CMN|比较两个数的相反数|
|CMP|32位数比较|
|EOR|32位逻辑异或|
|LDC LDC2|从协处理器取一个或多个32位值|
|LDM|从内存送多个32位字到ARM寄存器|
|LDR|从虚拟地址取一个单个的32位值|
|MCR MCR2 MCRR|从寄存器送数据到协处理器|
|MLA|32位乘累加|
|MOV|传送一个32位数到寄存器|
|MRC MRC2 MRRC|从协处理器传送数据到寄存器|
|MRS|把状态寄存器的值送到通用寄存器|
|MSR|把通用寄存器的值传送到状态寄存器|
|MUL|32位乘|
|MVN|把一个32位数的逻辑“非”送到寄存器|
|ORR|32位逻辑或|
|PLD|预装载提示指令|
|QADD|有符号32位饱和加|
|QDADD|有符号双32位饱和加|
|QSUB|有符号32位饱和减|
|QDSUB|有符号双32位饱和减|
|RSB|逆向32位减法|
|RSC|带进位的逆向32法减法|
|SBC|带进位的32位减法|
|SMLAxy|有符号乘累加(16位x16位)+32位=32位|
|SMLAL|64位有符号乘累加((32位x32位)+64位=64位)|
|SMALxy|64位有符号乘累加((32位x32位)+64位=64位)|
|SMLAWy|号乘累加((32位x16位)>>16位)+32位=32位|
|SMULL|64位有符号乘累加(32位x32位)=64位|
|SMULxy|有符号乘(16位x16位=32位)|
|SMULWy|有符号乘(32位x16位>>16位=32位)|
|STC STC2|从协处理器中把一个或多个32位值存到内存|
|STM|把多个32位的寄存器值存放到内存|
|STR|把寄存器的值存到一个内存的虚地址内间|
|SUB|32位减法|
|SWI|软中断|
|SWP|把一个字或者一个字节和一个寄存器值交换|
|TEQ|等值测试|
|TST|位测试|
|UMLAL|64位无符号乘累加((32位x32位)+64位=64位)|
|UMULL|64位无符号乘累加(32位x32位)=64位|
