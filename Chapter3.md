# ics17_detailed_fallibility
Some worst details that I've encountered in ICS (pku17)

Note: based on CS:APP(3eZN), Intel 64&IA32 Architectures Sofware Developer's Manual

## Chapter 3  程序的机器级表示 (work in progress)

### 1. 寄存器、数据格式

* 注意区分寄存器 与 **通用目的寄存器（整数寄存器）** 

* 整数寄存器表，注意存储器的不同用处

* 生成1、2个字节的指令不会改变寄存器高位的内容，生成4字节的指令会将高位置为0


### 2. 顺序指令

#### a. x86-64操作数

Imm(rb,ri,s)

要求：

* rb和ri都是**64位**寄存器
* s必须是1 2 4 8
* 不同指令允许的立即数范围不同（参考之后遇到的movabsq的用法）


***待考证***

> 有传言说 `lea (%rax,%rsp,8),%rax` 是不被允许的，因为ri不能是%rsp


### 3. 控制

### 4. 过程

### 5. 数组与结构

### 6. 程序行为
