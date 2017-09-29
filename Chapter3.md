# ics17_detailed_fallibility
Some worst details that I've encountered in ICS (pku17)

Note: based on CS:APP(3eZN), Intel 64&IA32 Architectures Sofware Developer's Manual

***


## Chapter 3  程序的机器级表示 (work in progress)

### 1. 寄存器、数据格式

* 注意区分寄存器 与 **通用目的寄存器（整数寄存器）** 

* 整数寄存器表，注意存储器的不同用处

* **生成1、2个字节的指令不会改变寄存器高位的内容，生成4字节的指令会将高位置为0**


***


### 2. 顺序指令

#### a. x86-64操作数

Imm(rb,ri,s)

要求：

* rb和ri都是**64位**寄存器 （可以是非整数寄存器，例如%rip，参考§3.6.5的汇编代码）
* s必须是1 2 4 8
* 不同指令允许的立即数范围不同（参考之后遇到的movabsq的用法）


***待考证***

> 有传言说 `lea (%rax,%rsp,8),%rax` 是不被允许的，因为ri不能是%rsp


#### b. 数据传送指令mov


后缀：**b w l q** *z s*。例如：mov*z* **wl** 将做了零拓展的字传送到双字

传送指令的两个操作数不能都是内存位置，并且寄存器是16个整数寄存器

> 因此，`movq (%rax),(%rbx)`，`movq 8(%rsp),%rip`，均不可以。

常规的movq指令只能以表示为**32位*补码***的立即数作为源操作数，而movabsq能以任意的64位立即数作为源操作数，但**目标只能是整数寄存器**

> 因此，`movq $0x123456789AB,%rax`，`movq $0x123456789AB,16(%rdi)`，均不可以。


零拓展支持的操作：

| Src size | Tgt Size Allowed           |
|---------:|----------------------------|
| 1        | 2 4 8                      |
| 2        | 4 8                        |

* 这里，不存在movzlq（4->8）的指令，而实际上movl已经可以完成，因为**生成4字节的指令会将高位置为0**


符号拓展支持的操作：

| Src size | Tgt Size Allowed           |
|---------:|----------------------------|
| 1        | 2 4 8                      |
| 2        | 4 8                        |
| 4        | 8                          |

* 特殊的符号拓展：cltq （%eax向%rax符号拓展）



#### c. lea指令与算术运算指令

#TODO work_in_progress


***

### 3. 控制

### 4. 过程

### 5. 数组与结构

### 6. 程序行为
