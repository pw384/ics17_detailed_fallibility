# ics17_detailed_fallibility
Some worst details that I've encountered in ICS (pku17)

Note: based on CS:APP(3eZN), Intel 64&IA32 Architectures Software Developer's Manual

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
* inc和dec指令**不改变**CF

<br /><br />

#### b. 条件跳转

#TODO

<br /><br />

***

### 4. 过程

### 5. 数组与结构

### 6. 程序行为
