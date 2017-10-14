# ics17_detailed_fallibility
Some worst details that I've encountered in ICS (pku17)

Note: based on CS:APP(3eZN), Intel 64&IA32 Architectures Software Developer's Manual

Mainly focused on x86-64

***


## Chapter 3  程序的机器级表示 (work in progress)

### 1. 寄存器、数据格式

* 注意区分寄存器 与 **通用目的寄存器（整数寄存器）**
* 整数寄存器表，注意存储器的不同用处
* **生成1、2个字节的指令不会改变寄存器高位的内容，生成4字节的指令会将高位置为0**

<br /><br />

***


### 2. 顺序指令

#### a. x86-64操作数

`Imm(rb,ri,s)`

要求：  
* rb和ri都是**64位**寄存器 （可以是非整数寄存器，例如%rip，参考§3.6.5的汇编代码）
* s必须是1 2 4 8
* 不同指令允许的立即数范围不同（参考之后遇到的movabsq的用法）

***待考证***

> 有传言说 `lea (%rax,%rsp,8),%rax` 是不被允许的，因为ri不能是%rsp

<br /><br />

#### b. 数据传送指令mov


后缀：**b w l q** *z s*。例如：mov*z* **wl** 将做了零拓展的字传送到双字  
传送指令的两个操作数不能都是内存位置，并且寄存器是16个整数寄存器  
> 因此，`movq (%rax),(%rbx)`，`movq 8(%rsp),%rip`，均不可以。

常规的movq指令只能以表示为**32位补码**的立即数作为源操作数，而movabsq能以任意的64位立即数作为源操作数，但**目标只能是整数寄存器**  
> 因此，`movq $0x123456789AB,%rax`，`movabsq $0x123456789AB,16(%rdi)`，均不可以。

<br />

零拓展支持的操作：

| Src size | Tgt Size Allowed           |
|---------:|----------------------------|
| 1        | 2 4 8                      |
| 2        | 4 8                        |

* 这里，不存在movzlq（4->8）的指令，而实际上movl已经可以完成，因为**生成4字节的指令会将高位置为0**
* *事实上，movzbq（1->8）、movzwq（2->8）似乎也没有什么存在的必要，因为movzbl/movzwl生成了四字节，把高位设为0*

<br />

符号拓展支持的操作：

| Src size | Tgt Size Allowed           |
|---------:|----------------------------|
| 1        | 2 4 8                      |
| 2        | 4 8                        |
| 4        | 8                          |

* 特殊的符号拓展：cltq （%eax向%rax符号拓展）

<br /><br />

#### c. lea指令与算术运算指令

注意操作数/比较顺序

lea指令与mov指令的不同？  
* lea S,D 将S算出的内存的地址传入D，即 * D = S
* mov S,D 将S算出的内存的内容传入D，即 * D = * S

inc, dec, neg, not（一元）、add, sub, imul, xor, or, and（二元）中，源和目的都可以是内存地址

sal, shl, sar, shr（二元）中，第一个操作数（源）只能是立即数，或者是%cl。并且在x86-64中，如果是w位长度的移位操作，那么位移量只由%cl的低w个位确定。一元移位运算默认位移量为1

**用sar做除法与idiv并不等价**
>Using the SAR instruction to perform a division operation does not produce the same result as the IDIV instruction.
The quotient from the IDIV instruction is rounded toward zero, whereas the “quotient” of the SAR instruction is
rounded toward negative infinity.

`xorq %rax,%rax` 常常用来初始化一个值为0

cs:app 3eZN 在133页的图3-12中有两处错误，emmmm
> clto 应为 **cqto**  
> 最后一行R[%rdx] <- R[%rdx]:R[%rax] / S 应为 R[%r**a**x] <- R[%rdx]:R[%rax] / S

<br /><br />

***

### 3. 控制

#### a. 条件码

一般情况下，算术运算指令都改变条件码，但有以下注意需要的地方：
* lea不改变条件码（因为它在intel手册上不算作算术运算）  
* 所有逻辑操作都会把CF和OF设为0，包括test
* inc和dec指令**不改变**CF
* 移位操作会把CF设为最后一个被移出的位（无论左移还是右移）
> Shifts the bits in the first operand (destination operand) to the left or right by the number of bits specified in the
second operand (count operand). Bits shifted beyond the destination operand boundary **are first shifted into the CF
flag**, then discarded. At the end of the shift operation, the CF flag contains the last bit shifted out of the destination
operand.

* 移位操作对于OF的行为，在CS:APP(3e, 3eZN)上的说法与Intel手册不同，Intel手册给出的规则略复杂，下面给出，仅供参考：
>对于移位操作，溢出标志设置为0。 (CS:APP 3eZN p. 136)

vs.

>The OF flag is affected **only on 1-bit shifts**. For left shifts, the OF flag is set to 0 if the most-significant bit of the
result is the same as the CF flag (that is, the top two bits of the original operand were the same); otherwise, it is
set to 1. For the SAR instruction, the OF flag is cleared for all 1-bit shifts. For the SHR instruction, the OF flag is set
to the most-significant bit of the original operand. (Intel ASDM vol. 2B Chapter 4.3 p.2B-4-581)

```
IF (COUNT and countMASK) = 1
  THEN
    IF instruction is SAL or SHL
      THEN
        OF ← MSB(DEST) XOR CF;
      ELSE
        IF instruction is SAR
          THEN
            OF ← 0;
          ELSE (* Instruction is SHR *)
            OF ← MSB(tempDEST);
        FI;
    FI;
  ELSE IF (COUNT AND countMASK) = 0
    THEN
      All flags unchanged;                //移动0位不改变flags
    ELSE (* COUNT not 1 or 0 *)
      OF ← undefined;                     //移动超过1位的，OF是未定义的
    FI;
FI;
```


<br /><br />

#### b. 条件跳转与条件传送

set指令
* set指令只设置目标的低字节，或是内存中的一个地址，因此当目标是寄存器的时候，之后会跟上零拓展以清除高位
* 图3-14

j指令
* j在机器级的编码中，实际跳转地址是之后的参数+PC，例如  
```
4004d0: 48 89 f8			mov		%rdi,%rax
4004d3: eb 03					jmp		4004d8<loop+0x8>
4004d5: 48 d1 f8			sar		%rax
4004d8: 48 85 c0			test	%rax,%rax
4004db: 7f f8					jg		4004d5<loop+0x5>
4004dd: f3 c3					repz retq
```
程序进行到第五行时，开始跳转，跳转位置为0xf8(-8)+**0x4004dd**=0x4004d5，注意当程序走到第五行进行跳转之前，PC指向该指令的下一条指令，即第六行的开头
* rep是啥？空操作，但可以使得处理器正确预测之后ret指令的目的，加快速度（3eZN p141）

有时使用条件传送而不使用条件跳转的原因 p. 146

条件跳转的副作用：p. 148



<br /><br />

#### c. 循环与开关

C语言中，`for(INIT;TEST;ITER) BODY;` 与 `INIT;while(TEST){BODY;ITER;}` 并不等价，因为for循环中continue指令并不会跳过 `ITER`

<br /><br />

***

### 4. 过程

#### a. pushq/popq

`pushq %rsp` 指令会把%rsp的旧值压到栈上（虽然这条指令很奇怪。。。）  
`popq %rsp` 指令会把栈顶端的值扔进%rsp里（虽然这条指令更奇怪。。。）

<br /><br />

#### b. 控制转移

注意图3-25（csapp3eZN）  
返回地址、提供给被调用者的参数，都算在调用者的栈帧中

栈帧对齐太复杂了，不展开了（应该不会考吧。。。书上的东西编译出来也不一样。。。

<br /><br />

#### c. 参数传递

x86-64中，函数的第二个参数存在%rsi中 ← 该说法错误，应为x86-64中，函数的第二个**整型**参数存在%rsi中。（考虑`func(double x1, int x2, int x3)`，x1 -> %xmm0，x2 -> %rdi，x3 -> %rsi。

注意参数入栈顺序，压栈先压后面的参数（注意图3-25）

x86-64中，栈上**传参**按8对齐

*struct如何传参？*

<br /><br />

#### d. 局部变量

x86-64中，栈上局部变量**不一定**按8对齐

栈上的局部变量的组织方式...很复杂的样子，拿下面的代码试一下效果

```c
// 全是整数
// 会发现在内存中的地址从高到低为（从栈底到栈顶为）：&x7>&x5>&x1>&x8>&x6>&x2>&x3>&x4
long x1=1;
int x2=2;
short x3=3;
char x4=4;
long x5=5;
int x6=6;
long x7=7;
int x8=8;
printf("%x %x %x %x %x %x %x %x\n",&x1,&x2,&x3,&x4,&x5,&x6,&x7,&x8);
printf("%x %x %x %x %x %x %x %x\n",&x7,&x5,&x1,&x8,&x6,&x2,&x3,&x4);
```

```c
// 有数组，发现数组按基本类型大小的排序和正常整数（上例）按大小的排序是反的
long x1=1;
int x2=2;
short x3=3;
char x4=4;
long x5=5;
int x6=6;
int x7[13];
char x8[13];
long x9[5];
printf("%x %x %x %x %x %x %x %x %x\n",&x1,&x2,&x3,&x4,&x5,&x6,&x7,&x8,&x9);
```

```c
// 同类型的数组，发现小数组靠近栈顶（地址小），起始地址 似 乎 都是16的倍数
char x1[2]={'a','b'};
char x2[2]={'c','d'};
char x3[3]={'e','f','g'};
char x4[7]={'h','i','j','k','l','m','n'};
char x5[6]={'o','p','q','r','s','t'};
char x6[5]={'u','v','w','x','y'};
char x7[13]={'z','0','1','2','3','4','5','6','7','8','9','A','B'};
char x8[13]={'C','D','E','F','G','H','I','J','K','L','M','N','O'};
char x9[5]={'P','Q','R','S','T'};
printf("%x %x %x %x %x %x %x %x %x\n",&x1,&x2,&x3,&x4,&x5,&x6,&x7,&x8,&x9);
for(int i=0;i<128;i++)
	printf("(%d,%c) ",i,x1[i]);
```

```c
// 然而这时x2的地址就不是16的倍数了哈哈哈
char x1[2]={'a','b'};
char x2[1]={'c'};
char x3[3]={'e','f','g'};
char x4[7]={'h','i','j','k','l','m','n'};
char x5[6]={'o','p','q','r','s','t'};
char x6[5]={'u','v','w','x','y'};
char x7[13]={'z','0','1','2','3','4','5','6','7','8','9','A','B'};
char x8[13]={'C','D','E','F','G','H','I','J','K','L','M','N','O'};
char x9[5]={'P','Q','R','S','T'};
printf("%x %x %x %x %x %x %x %x %x\n",&x1,&x2,&x3,&x4,&x5,&x6,&x7,&x8,&x9);
for(int i=0;i<128;i++)
	printf("(%d,%c) ",i,x2[i]); // 这里从x2开始输出
printf("\n");
```

（行吧太复杂了

定长数组直接压在栈上，变长数组在栈上增长，通过c的malloc（c++中是new）创造的内存区域位于堆上

<br /><br />

***

### 5. 数组与结构

#### a. 定长数组

#TODO

#### b. 变长数组与变长栈帧

#### c. 结构与联合

***

### 6. 程序行为
