## 简单说明

- 环境推荐用52破解虚拟机https://www.52pojie.cn/thread-661779-1-1.html
- 教程我是直接学文末的各种笔记文章，具体看个人基础来选择学习方法。
- 吾爱有官方教学视频可在爱盘找到，还有一些例子可用
- 写c的话ide推荐用vc6别用vs，vs会对代码进行优化。
- 这里学的汇编语言是32位的，虽然现在大部分计算机都是64位的，但从本质上来说64位也是从32位衍生的，只有你学好了32位才能更容易的去学64位。
- 每学一个东西都要搞清楚学习它的意义。学了干嘛？要学到什么程度？比如这里的汇编，就我而言为了搞二进制安全而学。目的是为了后面更好的杀软对抗和apt样本分析之类的。那么只需要快速的过一下，能达到使用od够用就行。要是遇到不会的指令直接去查即可，没必要像在学校里面一样，还敲敲几个汇编程序出来。在工作中更需要的是效率。
- 如果老哥是为了逆向之类的，那么汇编还是得再学深入点。





## 汇编基操

#### 数据宽度

数据长度也叫数据宽度，**超出最多宽度的数据会被丢弃掉**

数据宽度单位：

| 名称               | 大小                                                         |
| ------------------ | ------------------------------------------------------------ |
| 位（BIT）          | █（1位）                                                     |
| 字节（Byte）       | █\|█\|█\|█\|█\|█\|█\|█（8位）                                |
| 字（Word）         | █\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█（16位、2个字节） |
| 双字（Doubleword） | █\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█\|█（32位、2个字、4个字节） |

#### 无符号数、有符号数

无符号数、有符号数、原码、反码、补码、左移、右移、位运算、高位、低位

简单理解：**无符号数存储在计算机内就是其本身的值**，1就是1。而有符号数比如-1有一个符号位。个人理解。





#### 通用寄存器

通用寄存器可以往里面存储任意数据和值。寄存器分两种一种就是存值的，如下前四个。另一种用来存放内存地址的，也就是指针寄存器。**此处重点掌握**

| 通用寄存器 |      |        |                                  |
| ---------- | ---- | ------ | -------------------------------- |
| 32位       | 16位 | 8位    | 作用                             |
| EAX        | AX   | AL、AH | 存值，如变量、返回值、计算结果等 |
| ECX        | CX   | CL、CH | 存值                             |
| EDX        | DX   | DL、DH | 存值                             |
| EBX        | BX   | BL、BH | 存值                             |
| ESP        | SP   |        | 栈顶指针，当前正在使用的堆栈地址 |
| EBP        | BP   |        | 栈底指针，当前正在使用的堆栈地址 |
| ESI        | SI   |        | 源变址寄存器，放的是内存地址     |
| EDI        | DI   |        | 目的变址寄存器，放的是内存地址   |

除了通用寄存器还有一个特殊的指针寄存器**EIP**，它里面存放的值是CPU下次要执行的指令地址。汇编是从上到下，依次执行。



#### 内存地址的5种形式

##### mov

向内存中添加数据或从内存中获取数据

- 格式：MOV 目标操作数，源操作数
- 含义：将源操作数传送到目标操作数

```asm
MOV DOWRD PTR DS:[内存地址], 立即数
MOV DOWRD PTR DS:[内存地址], 32位通用寄存器
MOV 32位通用寄存器, DOWRD PTR DS:[内存地址]
```

DOWRD为数据宽度,存储的数据需要与DOWRD数据宽度一致.还可以用上面提到的BYTE、WORD

```asm
// 读取内存的值
mov eax, dword ptr ds:[0x13FFC4]
// 向内存中写入数据
mov dword ptr ds:[0X13FFC4], eax
```

```asm
// 读取内存的值
move ecx, 0x13FFD0
mov eax, dword ptr ds:[ecx]

// 向内存中写入数据
mov edx, 0x13FFD8
mov dword ptr ds:[edx], 0x87654321
```

```asm
// 读取内存的值
mov ecx, 0x13FFD0
mov eax, dword ptr ds:[ecx+4]

// 向内存中写入数据
mov edx, 0x13FFD8
mov dword ptr ds:[edx+0xC], 0x87654321
```

```asm
// 读取内存的值
mov eax, 0x13FFC4
mov ecx, 0x2
mov edx, dword ptr ds:[eax+eax*4]

// 向内存中写入数据
mov eax, 0x13FFC4
mov ecx, 0x2
mov dword ptr ds:[eax+eax*4], 0x87654321
```

```asm
// 读取内存的值
mov eax, 0x13FFC4 mov ecx, 0x2
mov edx, dword ptr ds:[eax+eax*4+4]

// 向内存中写入数据
mov eax, 0x13FFC4
mov ecx, 0x2
mov dword ptr ds:[eax+eax*4+4], 0x87654321
```



# 常用汇编指令

我们常用的汇编指令有：MOV、ADD、SUB、AND、OR、XOR、NOT

如下格式举例中表示含义：

含义

- r 代表通用寄存器

- m 代表内存

- imm 代表立即数

- r8 代表8位通用寄存器

- m8 代表8位内存

- imm8 代表8位立即数

- 其他以此类推



## MOV指令

表示**数据传送**，其格式为：

```asm
// MOV 目标操作数，源操作数
// 含义：将源操作数传送到目标操作数
MOV r/m8,r8
MOV r/m16,r16
MOV r/m32,r32
MOV r8,r/m
MOV r16,r/m16
MOV r32,r/m32
MOV r8, imm8
MOV r16, imm16
MOV r32, imm32
```

## ADD指令

表示**数据相加**，其格式为：

```asm
// ADD 目标操作数，源操作数
// 含义：将源操作数与目标操作数相加，最后结果给到目标操作数
ADD r/m8, imm8
ADD r/m16,imm16
ADD r/m32,imm32
ADD r/m16, imm8
ADD r/m32, imm8
ADD r/m8, r8
ADD r/m16, r16
ADD r/m32, r32
ADD r8, r/m8
ADD r16, r/m16
ADD r32, r/m32
```

## SUB指令

表示**数据相减**，其格式为：

```asm
// SUB 目标操作数，源操作数
// 含义：将源操作数与目标操作数相减，最后结果给到目标操作数
SUB r/m8, imm8
SUB r/m16,imm16
SUB r/m32,imm32
SUB r/m16, imm8
SUB r/m32, imm8
SUB r/m8, r8
SUB r/m16, r16
SUB r/m32, r32
SUB r8, r/m8
SUB r16, r/m16
SUB r32, r/m32
```

## AND指令

表示**数据相与（位运算知识）**，其格式为：

```asm
// AND 目标操作数，源操作数
// 含义：将源操作数与目标操作数进行与运算，最后结果给到目标操作数
AND r/m8, imm8
AND r/m16,imm16
AND r/m32,imm32
AND r/m16, imm8
AND r/m32, imm8
AND r/m8, r8
AND r/m16, r16
AND r/m32, r32
AND r8, r/m8
AND r16, r/m16
AND r32, r/m32
```

## OR指令

表示**数据相或（位运算知识）**，其格式为：

```asm
// AND 目标操作数，源操作数
// 含义：将源操作数与目标操作数进行或运算，最后结果给到目标操作数
OR r/m8, imm8
OR r/m16,imm16
OR r/m32,imm32
OR r/m16, imm8
OR r/m32, imm8
OR r/m8, r8
OR r/m16, r16
OR r/m32, r32
OR r8, r/m8
OR r16, r/m16
OR r32, r/m32
```

## XOR指令

表示**数据相异或（位运算知识）**，其格式为：

```asm
// XOR 目标操作数，源操作数
// 含义：将源操作数与目标操作数进行异或运算，最后结果给到目标操作数
XOR r/m8, imm8
XOR r/m16,imm16
XOR r/m32,imm32
XOR r/m16, imm8
XOR r/m32, imm8
XOR r/m8, r8
XOR r/m16, r16
XOR r/m32, r32
XOR r8, r/m8
XOR r16, r/m16
XOR r32, r/m32
```

## NOT指令

表示**非（位运算知识）**，其格式为：

```asm
// NOT 目标操作数
// 含义：将源操作数进行非运算，最后结果给到目标操作数
NOT r/m8
NOT r/m16
NOT r/m32
```

## MOVS指令

表示**数据传送**，它与MOV的不同处在于，它可以将内存的数据传送到内存，但也仅仅能如此，其格式为：

```asm
// MOVS EDI指定的内存地址，ESI指定的内存地址
// 含义：将ESI指定的内存地址的数据传送到EDI指定的内存地址（使用MOVS指令时，默认使用的就是ESI和EDI寄存器），MOVS指令执行完成后ESI、EDI寄存器的值会自增或自减，自增或自减多少取决于传送数据的数据宽度
MOVS BYTE PTR ES:[EDI], BYTE PTR DS:[ESI] //简写为：MOVSB
MOVS WORD PTR ES:[EDI], WORD PTR DS:[ESI] //简写为：MOVSW
MOVS DWORD PTR ES:[EDI], DWORD PTR DS:[ESI] //简写为：MOVSD
```

**MOVS指令举例说明**：

1. 先将ESI、EDI的值修改为对应内存地址

```asm
MOV ESI, 0x12FFC4
MOV EDI, 0x12FFD0
```

2. 将0x11223344存入EDI指定的内存地址中

```asm
MOV DWORD PTR DS:[ESI], 0x11223344
```

## STOS指令

表示**将AL/AX/EAX的值储存到EDI指定的内存地址**，其格式为：

```asm
// STOS EDI指定的内存地址
// 含义：将AL/AX/EAX的值储存到EDI指定的内存地址，STOS指令执行完成后EDI寄存器的值会自增或自减，自增或自减多少取决于传送数据的数据宽度，与MOVS指令一样自增或自减取决于DF位
STOS BYTE PTR ES:[EDI] ``//简写为：STOSB
STOS WORD PTR ES:[EDI] ``//简写为：STOSW
STOS DWORD PTR ES:[EDI] ``//简写为：STOSD
```

## REP指令

表示**循环**，其格式为：

```asm
// REP MOVS指令/STOS指令
// 含义：循环执行MOVS指令或STOS指令，循环次数取决于ECX寄存器中的值，每执行一次，ECX寄存器中的值就会减一，直至为零，REP指令执行完成
REP MOVSB
REP MOVSW
REP MOVSD
REP STOSB
REP STOSW
REP STOSD
```

## PUSH指令

表示**压入数据**，其格式为：

```asm
// PUSH 通用寄存器/内存地址/立即数
// 含义：向堆栈中压入数据，压入数据后会提升（sub）栈顶指针（ESP），提升多少取决于压入数据的数据宽度
PUSH r16/r32
PUSH m16/m32
PUSH imm8/imm16/imm32
```

## POP指令

表示**释放数据**，其格式为：

```asm
// POP 通用寄存器/内存地址
// 含义：释放压入堆栈中的数据，释放数据后会下降（add）栈顶指针（ESP），下降多少取决于释放数据的数据宽度
POP r16/r32
POP m16/m32
```



## JMP指令

表示**跳转**，其格式为：

```asm
// JMP 寄存器/内存/立即数
// 含义：JMP指令会修改EIP的值为指定的指令地址，也就修改了程序下一次执行的指令地址，我们也可以称之为跳转到某条指令地址。
```

## CALL指令

表示调用函数可理解为push+jmp，其格式为：

```asm
// CALL 寄存器/内存/立即数
// 含义：跟JMP指令的功能一样也可以修改EIP的值，不同的点说，CALL指令执行后会将其下一条指令地址压入堆栈，ESP栈顶指针的值减4
```

## RET指令

表示**返回**，配合call使用，让其执行完当前call之后回到之前调用位置继续执行，其格式为：

```asm
RET
// 含义：将当前栈顶指针的值赋给EIP，然后让栈顶指针加4</p>
```



# 常用的32汇编指令

和上面的有重复，不管了。懒得整理。

- ADD ：加法
- ADC ：带位加法
- SBB ：带位减法
- SUB：减法.
- INC ：加法.
- CMP ：比较.(两操作数作减法,仅修改标志位,不回送结果).
- AND ：与运算.
- OR ：或运算.
- XOR ：异或运算.
- NOT ：取反.
- MOV：传送字或字节.
- MOVSX：先符号扩展,再传送.
- PUSH：把字压入堆栈.
- POP：把字弹出堆栈.
- PUSHA：把AX,CX,DX,BX,SP,BP,SI,DI依次压入堆栈.
- POPA ： 把DI,SI,BP,SP,BX,DX,CX,AX依次弹出堆栈.
- PUSHAD ： 把EAX,ECX,EDX,EBX,ESP,EBP,ESI,EDI依次压入堆栈.
- POPAD ： 把EDI,ESI,EBP,ESP,EBX,EDX,ECX,EAX依次弹出堆栈.
- LEA : 装入有效地址. 例: LEA DX,0xAA //把0xAA地址存到DX.
- JMP ：无条件转移指令
- CALL：过程调用 .
- RET/RETF : 过程返回.

## JCC指令

| JCC指令     | 含义                                               | 英文                                                    | 检查符号位       | C语句                    |
| ----------- | -------------------------------------------------- | ------------------------------------------------------- | ---------------- | ------------------------ |
| JZ/JE       | 若为0则跳转；若相等则跳转                          | jump if zero;jump if equal                              | ZF=1             | if (i == j);if (i == 0); |
| JNZ/JNE     | 若不为0则跳转；若不相等则跳转                      | jump if not zero;jump if not equal                      | ZF=0             | if (i != j);if (i != 0); |
| JS          | 若为负则跳转                                       | jump if sign                                            | SF=1             | if (i < 0);              |
| JNS         | 若为正则跳转                                       | jump if not sign                                        | SF=0             | if (i > 0);              |
| JP/JPE      | 若1出现次数为偶数则跳转                            | jump if Parity (Even)                                   | PF=1             | /                        |
| JNP/JPO     | 若1出现次数为奇数则跳转                            | jump if not parity (odd)                                | PF=0             | /                        |
| JO          | 若溢出则跳转                                       | jump if overflow                                        | OF=1             | /                        |
| JNO         | 若无溢出则跳转                                     | jump if not overflow                                    | OF=0             | /                        |
| JC/JB/JNAE  | 若进位则跳转；若低于则跳转；若不高于等于则跳转     | jump if carry;jump if below;jump if not above equal     | CF=1             | if (i < j);              |
| JNC/JNB/JAE | 若无进位则跳转；若不低于则跳转；若高于等于则跳转； | jump if not carry;jump if not below;jump if above equal | CF=0             | if (i >= j);             |
| JBE/JNA     | 若低于等于则跳转；若不高于则跳转                   | jump if below equal;jump if not above                   | ZF=1或CF=1       | if (i <= j);             |
| JNBE/JA     | 若不低于等于则跳转；若高于则跳转                   | jump if not below equal;jump if above                   | ZF=0或CF=0       | if (i > j);              |
| JL/JNGE     | 若小于则跳转；若不大于等于则跳转                   | jump if less;jump if not greater equal jump             | SF != OF         | if (si < sj);            |
| JNL/JGE     | 若不小于则跳转；若大于等于则跳转；                 | jump if not less;jump if greater equal                  | SF = OF          | if (si >= sj);           |
| JLE/JNG     | 若小于等于则跳转；若不大于则跳转                   | jump if less equal;jump if not greater                  | ZF != OF 或 ZF=1 | if (si <= sj);           |
| JNLE/JG     | 若不小于等于则跳转；若大于则跳转                   | jump if not less equal;jump if greater                  | SF=0F 且 ZF=0    | if(si>sj)                |









## 汇编传参

汇编传参有两种方式，一种是丢寄存器里面，一种是丢堆栈里面。

当函数有很多参数的时候，不止8个，那我们使用通用寄存器去传参，明显不够用，所以我们需要使用堆栈帮助我们传递参数。



## 堆栈相关指令

- 堆栈相当于一个瓶子，往里面存东西。里面东西多了之后，找别的都需要用到最顶上的一个也就是esp寻址。

- 堆栈是两个东西，堆和栈。堆由程序员控制，如new一个对象，定义一个数组长度等等。栈完全由系统控制。

- 堆栈的地址使用是从**大用到小的（高位地址到低位地址）**。

- 堆栈讲究**先入后出**的概念

- 堆栈地址范围就可以是**ESP - EBP**

- 函数的参数从右往左的方式依次入栈，最后入栈的是函数的**返回地址**方便执行完当前call，回到之前的位置继续执行后面的指令。

  比如我有个函数fun1(int a, char b, int c )

  我在main函数中调用了fun1函数，

  那么参数入栈的时候

  最先入栈的是变量c

  其次入栈的是变量b

  接着入栈的是变量a

  最后入栈的是main函数调用这个fun1函数之后的地址。

  

### ESP寻址

借助的ESP去获取对应参数的地址，这种行为我们称之为ESP寻址。

```asm
mov eax, dword ptr ss:[esp+4]
```

`esp-4`就是有值推到堆栈里面，也就是往上抬。`esp+4`与之相反就是往下找具体地址，也就是esp寻址。

一个int占四个字节,所以就是esp的值减少了四。

如下在执行push 0x5，push 0xc和push0x9的操作的时候，esp就会相应的减少12，对应的十六进制就是10，所以ESP就从0019FEE4 – 10 = 0019FED8，也就是说如果要往栈里面存入数据，esp的值就是栈顶的地址值



![img](https://gitee.com/safe6/img/raw/master/t01b84262f82f0bdddf.jpg)





## 堆栈平衡

简单知道概览即可，这里暂不深入了解

需要注意的是这段代码只有Debug版本才会有，而在Release版本中堆栈的布局与这是不一样的。

```
00401094   add         esp,44h
00401097   cmp         ebp,esp
00401099   call        __chkesp (004010b0)
```

这段代码的意思就是对比esp和ebp是否一样，而我们知道堆栈在使用完成之后要恢复成员来的样子（**堆栈平衡**），所以在add指令之后ebp与esp应该是一样的，而后的call指令实际上就是调用了一个函数（**__chkesp**），这个函数就是用来**检查你的堆栈是否平衡**的。





## 标志寄存器

这里暂不深入了解





# References

- https://www.cnblogs.com/zimmerk/articles/2520011.html
- https://gh0st.cn/Binary-Learning/%E6%B1%87%E7%BC%96%E8%AF%AD%E8%A8%80.html
- https://www.anquanke.com/post/id/207594
- https://bbs.pediy.com/thread-262533.htm
- https://www.anquanke.com/post/id/260799#h2-4
