# 汇编进阶

## 标志寄存器

CPU 内部的寄存器中，有一种特殊寄存器，具有以下3种作用

1. 用来存储相关指令的某些执行结果
2. 用来为 CPU 执行相关指令提供行为依据
3. 用来控制 CPU 的相关工作方式

这类特殊的寄存器在 8086CPU 中，叫做*标志寄存器* 。8086CPU 的标志寄存器有16位，其中存储的信息通常被称为 *程序状态字(PSW)*,标志寄存器简称为falg

flag 和其他寄存器不同，其他寄存器用来存放数据，整个寄存器具有一个含义。而 falg 寄存器是```按位起作用的```,它的每一位都有专门的含义，记录特定信息

![flag](../images/x86asm/flag_reg.png)

flag de  1,3,5,12,13,14,15位在 8086CPU 中没有使用，不具有任何含义。而0,2,4，6,7,8,9,10,11 位都具有特殊的含义

### ZF 标志

falg 的第 6 位是 ZF,零标志位。它记录相关指令执行后，其结果是否为 0。如果结果为 0,那么 ZF=1，否则ZF=0

```x86asm
mov ax,1
sub ax,1
;执行后，结果为 0,ZF=1

mov ax,2
sub ax,1
;执行后，结果非 0,ZF=0
```

ZF 标记相关指令的计算结果是否为 0,如果为 0,则 ZF 要记录下 *是0* 这样的肯定信息。在计算机中 1 标识逻辑真，所以当
结果位 0 的时候 ZF=1,表示 *结果是0* 。若结果不为 0,则 ZF 要记录下 *不是0*，计算机中 0标识逻辑假，所以结果不为 0 时，
ZF=0,表示*结果不是0* 

```x86asm
mov ax,1
and ax,0
;执行后，结果为0,ZF=1,表示 结果是0

mov ax,1
or ax,0
;执行后，结果非0,ZF=0,表示 结果非0
```

> NOTE: 在8086CPU 的指令集中，有些指令的执行是影响标志寄存器的，eg:add,sub,mul,div,inc,or,and等，他们大多是运算指令(逻辑或算术运算)。有的指令的执行对 flag 没有影响，eg：mov,push,pop等，他们大多是传送指令。在使用一条指令的时候，要注意这条指令的全部功能，其中包括，执行结果对标志寄存器的哪些标志位造成影响

### PF 标志

flag 的第 2 位是 PF,奇偶标识为。它记录相关指令执行后，其结果的所有 bit 位中1 的个数是否为偶数。如果 1 的个数为偶数，PF=1,否则，PF=0

```x86asm
mov al,1
add al,10
;执行后，结果为 00001011B,有3个1,奇数，PF=0

mov al,1
or al,2
;执行后，结果为 00000011B,有2个1,偶数,PF=1

sub al,al
;执行后，结果为 00000000B,有0个1,偶数，PF=1
```

### SF 标志

flag 的第 7 位是 SF,符号标志位。记录相关指令执行后，结果是否为负。若为负，SF=1,否则，SF=0

计算机中通常用补码来表示有符号数据。计算机中的一个数据可以看作是有符号数，也可以看成无符号数。比如:

00000001B 可以看作无符号数1,或有符号数 +1

10000001B 可以看作无符号数129,或有符号数-127

也就是说，对于同一个二进制数据，计算机可以将其当作无符号数据来运算，也可以当作有符号数据来运算

```x86asm
mov al,10000001B
add al,1
;结果：(al) = 10000010B
```

可以将 add 指令进行的运算当作无符号数的运算，那么 add 指令相当于计算 129+1,结果为 130(10000010B);也可以将 add 指令进行的运算当作有符号数的运算，那么 add 指令相当于计算 -127+1,结果为 -126(10000010B)

不论我们如何看待，CPU 在执行 add 等指令时，就已经包含了两种含义，也将得到用同一种信息来记录的两种结果。关键在于我们需要哪一种结果

SF 标志，就是 CPU 对有符号数运算结果的一种记录，它记录数据的正负。在我们将数据当作有符号数来运算的时候，可以通过它来得知结果的的正负。如果将数据当作无符号数，SF 的值则没有意义，虽然相关的指令影响了它的值

也就是说，CPU 在执行 add 等指令时，是必然要影响到 SF 标志位的值的。至于我们需不需要这种影响，就看我们如何看待指令所进行的运算了

```x86asm
mov al,10000001B
add al,1
;执行后，结果为 10000010B,SF=1,
;如果指令进行的有符号数运算，则结果为负

mov al,10000001B
add al,01111111B
;执行后，结果为0,SF=0
;如果指令进行有符号数运算，则结果为 非负
```

### CF 标志

flag 的第 0 位是 CF,进位标志位。一般情况下，进行 *无符号数*运算时，CF 记录了运算结果的最高有效位 向更高位的进位值，或从更高位的借位值

对于为数为 N 的无符号数来说，其对应的二进制信息的最高位，即第 N - 1 位，就是它的最高有效位，而假想存在的第 N 位，就是相对于最高有效位的更高位

![CF](../images/x86asm/flag_reg_CF.png)

