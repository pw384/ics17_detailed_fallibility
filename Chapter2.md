# ics17_detailed_fallibility
Some worst details that I've encountered in ICS (pku17)

Note: based on CS:APP(3eZN), Intel 64&IA32 Architectures Sofware Developer's Manual

***

## Chapter 2  信息的表示

### 1. 整数运算

#### a. 运算优先级

| Level     |  Ops                                   |
|-----------|----------------------------------------|
| 9         | i++  i--                               |
| 8         | sizeof  ++i  --i  ~  !  -i  +i  (int)i |
| 7         | i * j  i/j  i%j                          |
| 6         | i+j  i-j                               |
| 5         | **i<<j**  **i>>j**                             |
| 4         | i<j  i<=j  i>j  i>=j                   |
| 3         | i==j  i!=j                             |
| 2.2       | i & j                                    |
| 2.1       | i \^ j                                    |
| 2.0       | i \| j                                    |
| 1.1       | i && j                                   |
| 1.0       | i \|\| j                                   |


#### b. 强制类型转化

Case 1: 

```c
unsigned x=1;
int y=2;
printf("%d\n",((x-2)<y));           // 0
printf("%d\n",((!x)-1<y));          // 1
	
```

一般情况下int和unsigned参与运算的话，有符号的会被隐含地转为无符号数。

**但是在C/C++语言中，!运算会把unsigned变为有符号数，所以上例第二行的输出是1。**

<br />

Case 2:

```c
printf("%d\n",((0-2)<0)+0u==0u); //0
```

即使外层是unsigned的比较，内层仍然是signed的比较，返回1，然后1强转成unsigned再比较，答案就是0

<br />

Case 3:

```c
#define INT_MAX 2147483647
#define INT_MIN -INT_MAX - 1
```

C规定字面值为一串数字\[+u/U\] \[+l/L\]，`-2147483648`并不是字面值，而是一个**字面值表达式**。

-2147483648最终的类型为：
| 字长 | ISO C90 | ISO C99 |
|--|--|--|
|32|unsigned|long long|
|64|long|long|

```c
int dcomp = (-2147483648 < 0); // 答案不固定
int hcomp = (0x80000000 < 0); // 永远是0
```

```c
int dtmin = -2147483648; 
int dcomp2 = (dtmin < 0); // 永远是1
int htmin = 0x80000000; 
int hcomp2 = (htmin < 0); // 永远是1
```

#### c. 整数运算没有单调性（例子太多了），但构成阿贝尔群


***


### 2. 浮点运算

#### a. 由于精度问题导致的...

* 无结合律
* 无交换律
* 无分配律
* *但是仍然有单调性*


**double不一定比float精确**

  例如：x是最小的double非规格化数，x-x/2-x/2就不等于0，但是fx=(float)x，fx-fx/2-fx/2就等于0（因为一直都是0）
  
  ps. 据说考试考过...？？？这也太坑了吧


#### b. NaN & inf:

* +INF + +INF -> +INF
* -INF + -INF -> -INF
* +INF - +INF -> NaN
* x!=x成立当且仅当x==NaN


#### c. 舍入

留双舍入的情况：

* int -> float
* 乘法运算时导致的截断


舍入到0的情况：
* 转为int
