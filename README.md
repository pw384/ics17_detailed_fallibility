# ics17_detailed_fallibility
Some worst details that I've encountered in ICS (pku17)

Note: based on CS:APP(3eZN), Intel 64&IA32 Architectures Sofware Developer's Manual

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
| 2.2       | i&j                                    |
| 2.1       | i^j                                    |
| 2.0       | i|j                                    |
| 1.1       | i&&j                                   |
| 1.0       | i||j                                   |


#### b. 强制类型转化

Case 1: 

`
	unsigned x=1;
	
	int y=2;
	
	printf("%d\n",((x-2)<y));           // 0
	
	printf("%d\n",((!x)-1<y));          // 1
	
`

一般情况下int和unsigned参与运算的话，有符号的会被隐含地转为无符号数。

**但是在C/C++语言中，!运算会把unsigned变为有符号数，所以上例第二行的输出是1。**


Case 2:

`
	printf("%d\n",(2147483647+1==2147483648));    //0, with compiler's warning: Integer overflow in expression 2147483647+1
	
	printf("%d\n",(2147483647+1==0x80000000));    //1, with compiler's warning: Integer overflow in expression 2147483647+1
`

字面值的特性...


Case 3:
`
  printf("%d\n",((0-2)<0)+0u==0u); //0
`

即使外层是unsigned的比较，内层仍然是signed的比较，返回1，然后1强转成unsigned再比较，答案就是0


Case 4:
`
	printf("%lld\n",2147483648);                  //2147483648
	
	printf("%lld\n",0x80000000);                  //2147483648
	
  	printf("%lld\n",100000000000);                //100000000000
	
	long x=100000000000;
	
	printf("%lld\n",x);                           //1215752192, with compiler's warning: overflow in implicit constant conversion
`

依旧是字面值的特性...


#### c. 整数运算没有单调性（例子太多了），但构成阿贝尔群

### 2. 浮点运算

#### a. 由于精度问题导致的...

无结合律

无交换律

无分配律

但是仍然有单调性

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


## Chapter 3  程序的机器级表示

先这样吧... 2017/9/29
