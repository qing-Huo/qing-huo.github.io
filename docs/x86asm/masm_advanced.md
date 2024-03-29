# 汇编进阶

## 数据处理的2个基本问题

计算机是进行数据处理、运算的机器，那么有两个基本的问题就包含在其中
1. 处理的数据在什么地方？
2. 要处理的数据有多长？

```x86asm
;bx si di bp 都是可以表示内存地址
;这4个寄存器可以单个出现，或只能以4种组合出现

;下面两行指令是错误的
mov ax,[bx + bp]
mov ax,[si + di]
```

> 只要在 [...] 中使用寄存器```bp```,而指令中没有显性给出段地址，则段地址默认在```ss```中

?> 在没有寄存器名存在的情况下，用操作符```X ptr```指明内存单元的长度，X 在汇编指令中可以为 word 或 byte。
在没有寄存器参与的内存单元访问指令中，用 word ptr 或 byte ptr *显性地指令要访问的内存单元的长度是必要* 
```x86asm
;下面的指令中，用 word ptr 指明了指令访问的内存单元是一个字单元
mov word ptr ds:[0],1
inc word ptr [bx]
inc word ptr ds:[0]
add word ptr [bx],2

;下面的指令，用 byte ptr 指令了指令访问的内存单元是一个字节单元
mov byte ptr ds:[0],1
inc byte ptr [bx]
inc byte ptr ds:[0]
add byte ptr [bx],2
```
## div 指令

div 是除法指令，用div做除法时应注意几个问题

1. 除数：有8位和16位两种
2. 被除数：默认放在 ax 或 dx和ax 中，如果除数为 8 位，被除数为16位，结果在ax中存放。如果除数为16位，被除数则为32位，在 dx 和 ax 中存放，dx 存放高16位，ax存放低16位
3. 结果：如果除数为 8 位，则 al 存储除法操作的商， ah 存储除法操作的余数。如果除数位 16 位，则 ax 存储除法操作的商， dx 存储除法操作的余数

```x86asm
div reg ; reg 表示一个普通寄存器(ax,bx,cx,dx等)
div 内存单元

div byte ptr ds:[0]
;(al) = (ax) / ((ds) * 16 + 0)的商
;(ah) = (ax) / ((ds) * 16 + 0)的余数

div word ptr es:[0]
;(ax) = [ (dx) * 10000H + (ax)] / ((es) * 16 + 0)的商
;(dx) = [ (dx) * 10000H + (ax)] / ((es) * 16 + 0)的余数

div byte ptr [bx + si + 8]
;(al) = (ax) / ((ds) * 16 + (bx) + (si) + 8)的商
;(ah) = (ax) / ((ds) * 16 + (bx) + (si) + 8)的余数

div word ptr [bx + si + 8]
;(ax) = [ (dx) * 10000H + (ax)] / ((ds) * 16 + (bx) + (si) + 8) 的商
;(dx) = [ (dx) * 10000H + (ax)] / ((ds) * 16 + (bx) + (si) + 8) 的余数 
```

- 用div 指令计算 100001 / 100

被除数 100001 大于 65535,只能用 ax 和 dx 联合存放 100001。即要进行 16 位的除法,除数100小于255,可以在
一个8位寄存器中存放，但是，因为被除数是 32 位的，除数应为16位，所以要用一个 16 位寄存器来存放除数 100

要分别位 dx 和 ax 赋值 100001 的高16位和低16位，所以应先将 100001 表示为 16进制形式：186A1H。程序如下

```x86asm
mov dx,1
mov ax,86A1H    ;(dx) * 10000H + (ax) = 100001
mov bx,64H  ; 64H = 100(ten)
div bx

;结果
;(ax) = 03e8H 即1000
;(dx) = 1   即余数1
```

- 用 div 计算 1001 / 100

```x86asm
mov ax,03e9H    ;03e9H == 10001
mov bl,64H   ;64H == 100
div bl

;结果
;(al) = 0AH 即10
;(ah) = 1   余数为1
```

### dd 伪指令

db 和 dw 定义字节型数据和字型数据。dd 用来定义 dword(double word,双字)型数据的。比如

```x86asm
data segment
    db 1    ;db 1 为01H,在data:0处，占1个字节
    dw 1    ;dw 1 为0001H,在data:1处，占1个字
    dd 1    ;dd 1 为00000001H,在data:3处，占2个字
data ends
```

用div 计算data 段中第一个数据除以第二个数据后的结果，商存储在第三个数据的存储单元内

```x86asm
assume cs:code,ds:data
data segment
	dd 100001
	dw 100
	dw 0
data ends

code segment
start:	mov ax,data
	mov ds,ax

	mov ax,ds:[0]	;ds:0 字单元中的低16位存储在 ax 中
	mov dx,ds:[2]	;ds:2 字单元中的高16位存储在 dx 中
	div word ptr ds:[4]	;用dx:ax 中的32位数据除以 ds:4 字单元中的数据
	mov ds:[6],ax	; 将商存储在 ds:6 字单元中

	mov ax,4c00H
	int 21H
code ends
end start
```

一组信息记录形式如下：

```x86asm
公司名称： DEC
总裁姓名：Ken Olsen
排    名：137
收    入：40
著名产品：PDP(小型机)
```
现在信息有了如下变化

```
Ken Olsen 的排名已上升至38
DEC 的收入增加至 70
该公司的著名产品已变为 VAX 系列计算机
```

修改内存中的过时数据

```x86asm
assume cs:codesg,ds:datasg
datasg segment
;为了在 debug 中直观显示，排名和收入写为了16进制
;以下4行数据在同一段，最后一行产品数据在第二段
	db 'DEC'	;公司名
	db 'Ken Olsen'	;总裁名
	dw 137H		;排名	=> 38
	dw 40H		;收入	=> 70
	db 'PDP'	;著名产品	=> 'VAX'
datasg ends

codesg segment
start:	mov ax,datasg
	mov ds,ax
	mov bx,0

	mov word ptr ds:[bx+12],38H 
	mov word ptr ds:[bx+14],70H

	mov si,0
	mov byte ptr ds:[bx+si+10H],'V'
	inc si
	mov byte ptr ds:[bx+si+10H],'A'
	inc si
	mov byte ptr ds:[bx+si+10H],'V'

	mov ax,4c00H
	int 21H
codesg ends
end start
```

### dup 

dup 是一个操作符，同 db、dw、dd 一样，也是由编译器处理的。他是和db、dw、dd等数据定义位指令配合使用，用来进行数据的重复

```x86asm
db 重复次数 dup (重复的字节型数据)
dw 重复次数 dup (重复的字型数据)
dd 重复次数 dup (重复的双字型数据)

db 3 dup (0)
;定义了3个字节，他们的值都是0，相当于 db 0,0,0

db 3 dup (0,1,2)
;定义了9个字节，他们是0,1,2,0,1,2,0,1,2 相当于 db 0,1,2,0,1,2,0,1,2

db 3 dup ('abc','ABC')
;定义了18个字节，他们是‘abcABCabcABCabcABC'

stack segment
    db 200 dup (0)
stack ends
;定义一个容量为200个字节的栈空间
```

### 实验7 结构化数据寻址

![struct](../images/x86asm/struct_seven_7.png)

```x86asm
assume cs:codesg
data segment
    db '1975', '1976', '1977', '1978', '1979', '1980', '1981', '1982', '1983' 
    db '1984', '1985', '1986', '1987', '1988', '1989', '1990', '1991', '1992'
    db '1993', '1994', '1995'
    ;表示21个年的21个字符串

    dd 16,22,382,1356,2390,8000,16000,24486,50065,97479,140417,197514
    dd 345980,590827,803530,1183000,1843000,2759000,3753000,4649000,5937000
    ;表示21个年公司总收入的21个dword型数据

    dw 3,7,9,13,28,38,130,220,476,778,1001,1442,2258,2793,4037,5635,8226
    dw 11542,14430,15257,17800
    ; 以上是表示21年公司雇员人数的21个word型数据
data ends

table segment
    db 21 dup ('year summ ne ?? ')
    ;div的余数不要，
table ends

codesg segment
start:  mov ax,data ;指向数据段
        mov ds,ax
        mov ax,table    ;指向结果段
        mov es,ax
        mov dx,0
        mov ax,0

        mov di,0
        ;循环时指向年份的指针
        mov si,21*4
        ;指向收入的指针
        mov bp,21*8
        ;指向人数的指针
        mov bx,0
        ;指向table段的指针

        mov cx,21
s:      push ds:[di]
        pop es:[bx]
        push ds:[di+2]
        pop es:[bx+2]
        ;将年份写入table段中的year处

        mov ax,ds:[si]
        mov es:[bx+5],ax
        mov dx,ds:[si+2]
        mov es:[bx+7],dx
        ;将收入写入table段中的summ处

        push ds:[bp]
        pop es:[bx+10]
        ;将人数写入 table 段的 ne 处

        div word ptr es:[bx+10] ;计算人均薪资
        mov es:[bx+13],ax   ; 将人均薪资赋值至 ?? 处

        add di,4    
        add si,4
        add bp,2
        add bx,16
        ;指向下一次循环的位置
        loop s

        mov ax,4c00H
        int 21H
codesg ends
end start
```

## 转移指令

```可以修改IP,或同时修改CS和IP的指令统称为转移指令```,转移指令就是可以控制CPU执行内存中某处代码的指令

- 8086CPU的转移行为有两类
    1. 只修改IP，时，称为段内转移，如：jmp ax
    2. 同时修改CS和IP时，称为段间转移，如：jmp 1000:0
- 由于转移指令对IP的修改范围的不同，段内转移又分为：短转移和近转移
    1. 短转移IP的修改范围为 -128～127
    2. 近转移IP的修改范围为-32768~32767
- 8086CPU的转移指令分为一下几类
    1. 无条件转移指令(jmp)
    2. 条件转移指令
    3. 循环指令(loop)
    4. 过程
    5. 中断

### offset 操作符

操作符 offset 在汇编中是由编译器处理的符号，它的功能是取得*标号*的偏移地址

```x86asm
assume cs:codesg
codesg segment
    start:	mov ax,offset start	;相当于 mov ax,0
   	    s:  mov bx,offset s		;相当于 mov bx,3
codesg ends
end start
```

### jmp

jmp 为无条件转移指令，可以只修改IP,也可以同时修改 CS 和 IP;jmp 要给出两种信息

1. 转移的目的地址
2. 转移的距离(段间转移、段内短转移，段内近转移)

#### 依据位移进行转移的jmp

```jmp short 标号```(转到标号处执行指令) ,这种格式的jmp 指令实现的是段内短转移，对IP 的修改范围为*-128～127* ,
向前时最多128个字节，向后转移时最多127个字节。其中```short```符号，说明指令进行的是短转移。

```x86asm
assume cs:codesg
codesg segment
start:  mov ax,0
        jmp short s
        add ax,1
s:      inc ax
codesg ends
end start
;最后ax 为1,执行 jmp short s 后，直接执行了 inc ax
```

```jmp short 标号```的功能为：(ip) = (ip) + 8位位移
1. 8位位移 = 标号处的地址 - jmp指令后的第一个字节的地址
2. short 指名此处的位移为8位位移
3. 8位位移的范围为*-128-127*,用补码表示 
4. 8位位移由编译程序在编译时计算

还有一种和```jmp short 标号```功能相近的指令格式，```jmp near ptr 标号```,他实现的是段内近转移

```jmp near ptr 标号```的功能为：(ip) = (ip) + 16位位移
1. 16位位移 = 标号处的地址 - jmp指令后的第一个字节的地址
2. near ptr 指名此处的位移为16位位移，进行的是段内近转移
3. 16位位移的范围为*-32768-32767* ,用补码表示
4. 16位位移由编译程序在编译时计算

#### jmp 段间转移

上面 jmp 指令，其对应的机器码中并没有转移的目的地址，而是相对于当前IP 的转移位移

```jmp far ptr 标号``` 实现的是段间转移，又称为远转移。功能如下：

```(CS) = 标号所在段的段地址; (IP) = 标号所在段中的偏移地址```; ```fat ptr``` 指令了指令用标号的段地址和偏移地址修改CS 和 IP

#### jmp 段内转移(内存地址)

转移地址在内存中的 jmp 指令有两种格式：

1. ```jmp word ptr 内存单元地址(段内转移)```; 从内存地址处开始存放着一个字，是转移的目的偏移地址
    * 内存单元地址可用寻址方式的任一格式给出
    ```
    mov ax,0123H
    mov ds:[0],ax
    jmp word ptr ds:[0]
    ;执行后，(IP) = 0123H

    mov ax,0123H
    mov [bx],ax
    jmp word ptr [bx]
    ;执行后，(IP) = 0123H
    ```
2. ```jmp dword ptr 内存单元地址(段间转移)```; 从内存地址开始处存放着2个字，高地址处的字是转移的目的段地址，低地址处是转移的目的偏移地址
    * (CS) = (内存地址 + 2)
    * (IP) = (内存地址)

    ```x86asm
    mov ax,0123H
    mov ds:[0],ax
    mov word ptr ds:[2],0
    jmp dword ptr ds:[0]
    ;(CS) = 0,(IP)= 0123H,CS:IP指向0000:0123

    mov ax,0123H
    mov [bx],ax
    mov word ptr [bx+2],0
    jmp dword ptr [bx]
    ;(CS) = 0,(IP) = 0123H,CS:IP指向 0000:0123H
    ```

- 定义data 段中的数据，使得 jmp 指令执行后，CS:IP 指向程序的第一条指令

```x86asm
assume cs:codesg

data segment
        db 0
        dw offset start ;也可写 0
data ends

codesg segment
start:  mov ax,data
        mov ds,ax
        mov bx,0
        jmp word ptr [bx+1]
codesg ends
end start
```

- 使用 Debug 查看内存，结果如下

```
2000:1000   BE 00 06 00 00 00 ...
```

则此时，CPU执行如下指令后，CS 和 IP 的值为？

```
mov ax,2000H
mov es,ax
jmp dword ptr es:[1000H]

; CS = 0006
; IP = 00BE
```

### jcxz 指令

jcxz 指令为有条件转移指令，```所有的有条件转移指令都是短转移，在机器码中包含转移的唯一，而不是目的地址```; 对 IP 的修改范围都为*-128-127* 

```jcxz 标号``` 如果 (CX) = 0,转移到标号处执行。当 (CX) = 0时，(IP) = (IP) + 8位位移
1. 8位位移 = 标号处的地址 - jcxz 指令后的第一个字节的地址
2. 8位位移的范围为 *-128～127*,用补码表示
3. 8位位移由编译程序在编译时计算
4. 当 (CX) != 0 时，什么也不做

```jcxz 标号``` 相当于 ```if ((cx) == 0) jmp short 标号```

- 利用 jcxz 指令，实现在内存 2000H 段中查找第一个值为 0 的字节，找到后，将其偏移地址存储在 dx 中

```x86asm
assume cs:codesg
codesg segment
start:  mov ax,2000H
        mov ds,ax
        mov bx,0

s:      mov ch,0    ;从这里开始往下的4行都是补全的程序
        mov cl,ds:[bx]
        jcxz ok
        inc bx

        jmp short s
ok:     mov dx,bx   ;此处的 ok标号没有意义,同下

        mov ax,4c00H
        int 21H

codesg ends
end start
```

### loop 指令

loop 指令为循环指令，```所有的循环指令都是短转移```,在对应的机器码中包含转移的位移，而不是目的地址，对IP的修改范围都是：*-128~127* 

```loop 标号``` ((CX) = (CX) - 1,如果(CX) != 0,转移到标号处执行)

```loop 标号``` 的功能相当于 ```(CX) --; if((CX) != 0) jmp short 标号;```

- 利用 loop 指令，实现在内存 2000H 段中查找第一个值为 0 的字节，找到后，将其偏移地址存储在 dx 中

```x86asm
assume cs:codesg
codesg segment
start:  mov ax,2000H
        mov ds,ax
        mov bx,0

s:      mov cl,ds:[bx]
        mov ch,0
        inc cx  ;避免cx-- 为FFFF 后产生的误差
        inc bx
        loop s

ok:     dec bx  ;dec 和 inc 相反,相当于 bx --
        mov dx,bx

        mov ax,4c00H
        int 21H
codesg ends
end start
```

#### 向显存写入三行字符串(实验9)

内存空间中，```B8000H~BFFFFH```共 32KB 的空间，为 80 X 25 彩色字符的显示缓冲区。向这个地址空间写入数据，写入的内容立即出现在显示器上

在 80 X 25 彩色字符模式下，显示器可以显示 25行，每行 80个字符，每个字符可以有 256种属性(背景色、前景色、闪烁、高亮等组合信息)

一个字符在显示缓冲区中要```占用2个字节```,分别存放字符的 ASCII码和属性。80 X 25模式下，一屏的内容在显示缓冲区中共占用 4000 个字节

显示缓冲区分为 8 页，每页 4KB(约等于4000B),显示器可以显示任意一页的内容。一般，显示第0页的内容。即，```B8000H~B8F9FH```中的 4000个字节的内容将出现在显示器上

在一页显示缓冲区中：

偏移 000~09F 对应显示器上的第一行(80个字符占 160个字节)

偏移 0A0~13F 对应显示器上的第二行

偏移 140~1DF 对应显示器上的第三行

依次类推，偏移 F00~F9F 对应显示器上的第25行

在一行中，一个字符站2个字节的存储空间(一个字),低位字节存储字符的 ASCII码，高位字节存储字符的属性。一行共80个字符，占160个字节

即在一行中

00~01 单元对应显示器的第一列

02~03 单元对应显示器的第二列

04~05 单元对应显示器的第三列

依次类推，9E~9F 对应显示器上的第 80 列

在显示缓冲区中，偶数地址存放字符，奇数地址存放字符的属性

一个在屏幕上显示的字符，具有前景(字符色)和背景(底色)两种颜色，字符还可以以高亮度和闪烁的方式显示。各种属性信息被记录在属性字节中

属性字节的格式：

```x86asm
0   0 0 0   0 0 0 0 
BL  R G B   I R G B
    背景
              前景
;BL=闪烁
;I=高亮
```

红底绿字，属性字节为：01000010B

红底闪烁绿字，属性字节为：11000010B

黑底白字，属性字节为：00000111B

白底蓝字，属性字节为：01110001B


```x86asm
assume cs:code
data segment
        db 'welcome to linux'
        ; 0000 0000
        ;  rgb  rgb
        ;  背景 前景
        ;闪烁  高亮
        db 00000010b    ;绿色
        db 00100100b    ;绿底红色
        db 01110001b    ;白底蓝色
data ends

code segment

start:  mov ax,data
        mov ds,ax
        mov bx,0    ;指向显存的字单元
        mov di,0    ;指向下一行显存的字单元
        mov bp,0    ;指向data定义数据的字符

        mov ax,0B810H
        mov es,ax   ;es 指向显存(屏幕)
        mov si,16   ;指向字符属性(共定义了3个字符属性)

        mov cx,3
showcol: mov ax,0    ;临时保存显存字符和字符属性
        push cx
        mov ah,ds:[si]  ;给高位赋值字符属性

        mov cx,16
showrow: mov al,ds:[bp]  ;低位赋值字符的ASCII码
        mov es:[bx+di],ax  ;向显存写入ASCII码及属性
        inc bp  ;指向下一个定义字符
        add bx,2    ;指向下一个显存的字单元

        loop showrow
        mov bp,0
        inc si
        mov bx,0
        add di,160
        pop cx

        loop showcol

        mov ax,4c00H
        int 21H
code ends
end start
```

## call & ret 

call & ret 都是*转移指令*,他们都*修改 IP*,或```同时修改 CS 和 IP```

### ret 和 retf

ret 指令用栈中的数据，修改 IP 的内容，从而实现```近转移```
retf 指令用栈中的数据，修改 CS 和 IP 的内容，从而实现```远转移```

CPU 执行 ret 指令时，进行两步操作

```x86asm
(IP) = ((ss) * 16 + (sp))
(sp) = (sp) + 2
;用汇编指令描述 ret 
;CPU 执行 ret 指令时，相当于
;pop IP
```

CPU 执行 retf 指令时，进行4步操作

```x86asm
(IP) = ((ss) * 16 + (sp))
(sp) = (sp) + 2
(CS) = ((ss) * 16 + (sp))
(sp) = (sp) + 2
;用汇编指令描述 retf
;CPU 执行 retf 指令时，相当于
;pop IP
;pop CS
```
- 下面的程序，ret 执行后，IP = 0, CS:IP 指向代码段的第一条指令

```x86asm
assume cs:code
stack segment
        db 16 dup (0)
stack ends
code segment
        mov ax,4c00H
        int 21H
start:  mov ax,stack
        mov ss,ax
        mov sp,16
        mov ax,0
        push ax
        mov bx,0
        ret

code ends
end start
```

- 下面的程序，retf 执行后, CS:IP 指向代码段的第一条指令

```x86asm
assume cs:code
stack segment
        db 16 dup (0)
stack ends
code segment
        mov ax,4c00H
        int 21H
start:  mov ax,stack
        mov ss,ax
        mov sp,16
        mov ax,0
        push cs
        push ax
        mov bx,0
        retf
code ends
end start
```

### call 指令

CPU 执行 call 指令时，进行2步操作

```x86asm
将当前的 IP 或 CS & IP 入栈
转移
```

call 指令不能实现短转移，除此之外，call 指令实现转移的方法和 jmp 指令的原理相同

### 依据位移进行转移的 call

call 标号(将当前的 IP 入栈后，转到标号处执行指令)

CPU 执行这种格式的 call 指令时，进行如下操作

```x86asm
(sp) = (sp) - 2
    ((ss) * 16 + (sp)) = (IP)
(IP) = (IP) + 16位位移

16位位移 = 标号处的地址 - call指令后的第一个字节的地址
16位位移的范围为 -32768～32767,用补码表示
16位位移由编译程序在编译时算出
```

如果用汇编语法来解释这种格式的 call 指令，则CPU 执行 *call 标号* 时，相当于如下

```x86asm
push IP
jmp near ptr 标号
```

- 下图的指令执行后，ax 中的数值为多少？

![call](../images/x86asm/call_first.png)

```x86asm
ax = 0
;call s == push IP && jmp near ptr s
执行 call s 后，IP 指向 inc ax,前两条指令字节长度为6
在 IP = 6后，被 push
之后 jmp near ptr 至 s 处，执行 pop ax
ax = 6
```

### 转移的目的地址在指令中的 call

前面的 call 指令，其对应的机器指令中并没有转移的目的地址，而是相对于当前 IP 的转移位移

```call far ptr 标号```实现的是*段间转移* ,CPU 执行这种格式的 call 指令时，进行如下操作

```x86asm
(sp) = (sp) -2
    ((ss) * 16 + (sp)) = (CS)
    (sp) = (sp) - 2
    ((ss) * 16 + (sp)) = (IP)
(CS) = 标号所在段的段地址
(IP) = 标号所在段的偏移地址 
```

用汇编语法来解释这种格式的 call 指令，则 CPU 执行 *call fat ptr 标号* 时，相当于：

```x86asm
push CS
push IP
jmp fat ptr 标号
```
- 下图的指令执行后，ax 中的数值为多少？

![call_second](../images/x86asm/call_second.png)

推导过程和上题相同，直接写结果：*ax = 1010H* 

### 转移地址在寄存器中的 call 指令

```x86asm
;指令格式
call 16位reg

(sp) = (sp) - 2
((ss) * 16 + (sp)) = (IP)
(IP) = (16位reg)
```

用汇编语法来解释这种格式的 call 指令，CPU 执行 *call 16位reg* 时，相当于

```x86asm
push IP
jmp 16位reg
```

- 下图的指令执行后，ax 中的数值为多少？

![call_three](../images/x86asm/call_three.png)

```x86asm
需要注意的是 [bp]
[bp] 的表示在本章开头有写明
单纯的 [bp] 中，是用 ss 来做段地址的
ax = 000BH
```

### 转移地址在内存中的 call

转移地址在内存中的 call 指令有2种格式

```x86asm
;1
call word ptr 内存单元地址
;用汇编语法解释这种格式的 call 指令，CPU 执行 call word ptr 内存地址 相当于进行
push IP
jmp word ptr 内存单元地址

;比如下面的指令
mov sp,10H
mov ax,0123H
mov ds:[0],ax
call word ptr ds:[0]
;执行后，IP = 0123H, SP = 0EH
```

```x86asm
;2
call dword ptr 内存地址单元
;用汇编语法来解释这种格式的call 指令，CPU 执行 call dword ptr 内存地址 详单与进行
push CS
push IP
jmp dowrd ptr 内存单元地址

;比如，下面的指令
mov sp,10H
mov ax,0123H
mov ds:[0],ax
mov word ptr ds:[2],0
call dword ptr ds:[0]
;执行后，CS = 0,IP = 0123H,sp = 0cH
```

- 检测点10.5

1. 下面的程序执行后，ax中的值位多少？

```x86asm
assume cs:code
stack segment
        dw 8 dup (0)
stack ends
code segment
start:  mov ax,stack
        mov ss,ax
        mov sp,16
        mov ds,ax
        mov ax,0
        call word ptr ds:[0eH]
        inc ax
        inc ax
        inc ax
        mov ax,4c00H
        int 21H
code ends
end start
``` 

ax 中的值为 3, ds 与 ss 中存放的段地址相同，在执行了 *call word ptr ds:[0eH]*后，程序会将 下一条指令 *inc ax* 的偏移量入栈，然后跳转到栈顶所指向的指令的位置，即跳转到第一条 inc ax 的位置， 故最后 ax 为 3。

> note: 在使用 debug 单步跟踪时，由于 t 命令所导致的中断，而影响了栈中的值

2. 下面的程序执行后，ax和bx中的数值为多少？

```x86asm
assume cs:codesg
data segment
	dw 8 dup (0)
data ends
code segment
start:	mov ax,data
	mov ss,ax
	mov sp,16
	mov word ptr ss:[0],offset s
	mov ss:[2],cs
	call dword ptr ss:[0]
	nop
s:	mov ax,offset s
	sub ax,ss:[0ch]     
   	mov bx,cs
	sub bx,ss:[0eh]
	mov ax,4c00h
	int 21h
code ends
end start
```

ax中的数值为1，bx中的数值为0，注意到程序的一开始将a的偏移量和cs放入ss:[0]和ss:[2]中，然后调用call指令，将CS和IP(nop指令的偏移量)依此压栈后跳转到s处继续执行，ax最终为s的偏移量减去nop指令所在位置的偏移量，为1，bx最终为cs的段地址相减，为0。

### mul 指令

mul 是乘法指令，使用 mul 做乘法的注意点如下：

1. 两数相乘：要么都是8位，要么都是16位。
    * 如果是8位，一个默认放在 al 中，另一个放在8位 reg 或内存字节单元中
    * 如果是16位，一个默认在 ax 中，另一个放在16位 reg 或内存字单元中
2. 结论：如果是8位乘法，结果默认放在 ax 中
    * 如果是 16位乘法，结果高位默认在 dx 中存放，低位在 ax 中存放

```x86asm
;mul 的格式
mul reg
mul 内存单元

;内存单元可以用不同的寻址方式给出
mul byte ptr ds:[0]
;含义如下
(ax) = (al) * ((ds) * 16 + 0)

mul word ptr [bx+si+8]
(ax) = (ax) * ((ds) * 16 + (bx) + (si) + 8)结果的低16位
(dx) = (ax) * ((ds) * 16 + (bx) + (si) + 8)结果的高16位
```

- 计算 100 * 10

```x86asm
; 100 和 10 小于 255,可以做 8位乘法
mov al,100
mov bl,10
mul bl
;结果： (ax) = 1000(03e8H)
```

- 计算 100 * 10000

```x86asm
; 100 小于 255, 可 10000 大于 255,所以做16位乘法
mov ax,100
mov bx,10000
mul bx
;结果：(ax) = 4240H,(dx) = 000FH, (F4240H=1000000(百万))
```

### 参数和结果传递

设计一个子程序，可以根据提供的 N,计算 N 的3次方

这里面有两个问题

1. 将参数 N 存储在什么地方？
2. 计算得到的数值，存储在什么地方？

这道题，可以用寄存器来存储，将参数放到 bx 中，由于子程序要计算 N**3,可以使用多个 mul 命令，为了方便，可以将结果
放到 dx 和 ax 中。子程序如下

```x86asm
;说明：计算 N 的3次方
;参数：(bx) = N
;结果：(ds:ax) = N**3
cube:   mov ax,bx
        mul bx
        mul bx
        ret
```

完成程序如下

```x86asm
assume cs:code
data segment
    dw 1,2,3,4,5,6,7,8
    dd 0,0,0,0,0,0,0,0
data ends
code segment
start:  mov ax,data
        mov ds,ax
        mov si,0
        mov di,16

        mov cx,8
s:      mov bx,ds:[si]
        call cube
        mov ds:[di],ax
        mov ds:[di+2],dx
        add si,2
        add di,4
        loop s

        mov ax,4c00H
        int 21H

cube:   mov ax,bx
        mul bx
        mul bx
        ret
code ends
end start
```
### 数据的批量传送

前面计算 N**3 的子程序 cube 只有一个参数，放在 bx 中，如果有多个参数呢？

```可以将批量参数放到内存中，然后将他们所在内存空间的首地址放在寄存器中，传递给需要的子程序。对于具有批量数据的返回结果，也用这种方法```

设计一个子程序，功能：将一个全是字母的字符串转为大写

解析：这个子程序需要知道两件事，字符串的内容和字符串的长度。因为字符串中的字母可能很多，所以可以将字符串在内存中的首地址
放在寄存器中传递给子程序。由于子程序中要用循环，循环的次数恰好是字符串的长度。可以将字符串的长度放在 cx 中

```x86asm
assume cs:code
data segment
        db 'conversation'
data ends
code segment
start:  mov ax,data
        mov ds,ax
        mov si,0    

        mov cx,12
        call capital

        mov ax,4c00H
        int 21H

capital:and byte ptr [si],11011111b ;将 ds:si 所指单元的字母转为大写
        inc si
        loop capital
        ret
code ends
end start
```

### 寄存器冲突的问题

设计一个子程序，功能：将一个全是字母，并以0结尾的字符串转为大写

由于题目要求字符串后面会有0,标记字符串的结束。所以可以用 jcxz 来检测0,遇到就返回。由于不知道循环次数，所以使用 jmp 指令来跳转执行

```x86asm
capital:push cx ;保存外部会使用的数据
        push si
change: mov cl,ds:[si]  ;如果(cx)=0,ret
        mov ch,0    ;否则转大写
        jcxz ok
        and byte ptr [si],11011111b ;将 ds:si 所指单元的字母转为大写
        inc si
        jmp short change
ok:     pop si  ;恢复外部数据并返回
        pop cx
        ret
```

将多个以 0 结尾的字符串转为大写

```x86asm
assume cs:code
data segment
;        db 'conversation',0
;        db 'hello'
真的
;        db 'conversation',0
        db 'word',0
        db 'unix',0
        db 'wind',0
        db 'good',0
        db 'linux',0
data ends
code segment
;将一个全是字母，以0结尾的字符串，转大写
;参数：ds:si指向字符串的首地址
;结果:没有返回值
start:  mov ax,data
        mov ds,ax
        mov si,0    

        mov cx,5
s:      call capital
        add si,5
        loop s

        mov ax,4c00H
        int 21H

capital:push cx
        push si
change: mov cl,ds:[si]  ;如果(cx)=0,ret
        mov ch,0    ;否则转大写
        jcxz ok
        and byte ptr [si],11011111b ;将 ds:si 所指单元的字母转为大写
        inc si
        jmp short change
ok:     pop si
        pop cx
        ret

code ends
end start
```

### 编写子程序

#### 向屏幕显示字符串

```x86asm
assume cs:code
data segment
        db 'Welcome to linux!',0
data ends
code segment
;子程序描述
;名称：show_str
;功能：在指定的位置，用指定的颜色，显示一个用 0 结束的字符串
;参数：(dh)=行号(取值0～24),(dl)=列号(取值0～79)
;      (cl)=颜色，ds:si指向字符串的首地址
;返回：无
;举例：在屏幕的8行3列，用绿色显示data段中的字符串
; row = 160, 第8行 == 160 * 7 == 460H
; columnes = +2，第3列 == 04~05
start:  mov dh,8    ;显存的行
        mov dl,3    ;现存的列
        mov cl,2    ;字符的属性
        mov ax,data
        mov ds,ax
        mov si,0
        call showStr

        mov ax,4c00H
        int 21H

showStr:push cx
        push si
        push ax
        push bx
        push dx

        mov ax,0B800H   ;显存起始地址
        mov es,ax   
        mov bx,0
        mov bl,dl
        inc bx  ;bx指向要写入的位置
        mov ah,cl

change: mov al,ds:[si]  ;用ax存储字符和属性
        mov ch,0
        mov cl,ds:[si]  ;用cx判断是否返回
        jcxz ok
        inc si  ;si指向data段的下一个字符
        mov es:[460H+bx],ax ;向显存写入数据
        add bx,2    ;移动指向显存的指针(指向下一个要写入的位置)
        jmp change

ok:     pop dx
        pop bx
        pop ax
        pop si
        pop cx
        ret

code ends
end start
```

#### 计算1000000 / 10 产生的溢出问题

```x86asm
assume cs:code,ss:stack,ds:data
data segment
        dd 1000000
data ends
stack segment
        db 128 dup (0)
stack ends
code segment
start:  mov ax,stack
        mov ss,ax
        mov sp,128

        mov bx,data
        mov ds,bx

        mov bx,0

        mov ax,ds:[bx+0]    ;参数 被除数低16位  L
        mov dx,ds:[bx+2]    ;参数 被除数高16位  H

        mov cx,10   ;参数 除数 N

        push ax
        mov bp,sp   ;ss:[bp+0]访问 ax 中的 L

        call longDiv

        mov ax,4c00H
        int 21

;======================================
;计算公式如下
;X/N = int(H/N)*65536 + [rem(H/N) * 65536+L] / N
;X 被除数，范围:[0,FFFFFFFF]
;N 除数，范围:[0,FFFF]
;H X的高16位，范围:[0,FFFF]
;L X的低16位，范围:[0,FFFF]
;int() 描述运算符，取商，ps:int(38/10)=3
;rem() 描述运算符，取余数，ps:int(38/10)=8

longDiv:mov ax,dx
        mov dx,0
        div cx  ;ax = int(H/N) => dx    dx = rem(H/N)*65536

        push ax

        mov ax,ss:[bp+0]    ;[rem(H/N)*65536+L]

        div cx  ;ax = [rem(H/N)*65536+L] / N 的商， dx = 余数

        mov cx,dx

        pop dx

        ret
code ends
end start
```
#### 将 data 段中定义的数据显示在屏幕上

```x86asm
assume cs:code,ds:data,ss:stack
data segment
	dw 123,12666,1,8,3,38
data ends

stack segment
	db 128 dup (0)
stack ends

string segment
        db 10 dup ('0'),0
string ends

code segment
start:	mov ax,stack
	mov ss,ax
	mov sp,128

    call clear_screen

    call init_reg

    call show_number

;===================================
show_number:
        mov bx,0
        mov si,9
        mov di,160*10+30*2

        mov cx,6

showNum:call show_word
        add di,160
        add bx,2
        loop showNum
        ret       

;===================================
show_word:
        push ax
        push bx
        push cx
        push dx
        push ds
        push es
        push si
        push di
        mov ax,ds:[bx]  ;16位除法
        mov dx,0

        call short_div

        call init_reg_show_string
        call show_string

        pop di
        pop si
        pop es
        pop ds
        pop dx
        pop cx
        pop bx
        pop ax
        ret
;===================================
show_string:
        push cx
        push ds
        push es
        push si
        push di
        mov cx,0

showString:
        mov cl,ds:[si]
        jcxz showStringRet
        mov es:[di],cl
        add di,2
        inc si 
        jmp showString

showStringRet:
        pop di
        pop si
        pop es
        pop es
        pop cx
        ret
;===================================
init_reg_show_string:
        mov bx,string
        mov ds,bx

        mov bx,0B800H
        mov es,bx
        ret
;===================================

short_div:
        mov cx,10
        div cx
        add dl,30H
        mov es:[si],dl
        mov cx,ax
        jcxz shortDivRet
        dec si
        mov dx,0
        jmp short_div

shortDivRet:
        ret
;===================================
init_reg:  
        mov bx,data
        mov ds,bx

        mov bx,string
        mov es,bx
        ret

;===================================

clear_screen:
        mov bx,0B800H
        mov es,bx

        mov bx,0
        mov dx,0700H
        mov cx,2000

clear:  mov es:[bx],dx
        add bx,2
        loop clear
        ret
;===================================

	mov ax,4c00H
	int 21H
code ends
end start
```
### 课程设计

将实验7中 Power idea 公司的数据按下图所示显示在屏幕上

![ten](../images/x86asm/ten.png)

```x86asm
assume cs:codesg,ds:data,ss:stack
data segment
    db '1975', '1976', '1977', '1978', '1979', '1980', '1981', '1982', '1983' 
    db '1984', '1985', '1986', '1987', '1988', '1989', '1990', '1991', '1992'
    db '1993', '1994', '1995'
    ;表示21个年的21个字符串

    dd 16,22,382,1356,2390,8000,16000,24486,50065,97479,140417,197514
    dd 345980,590827,803530,1183000,1843000,2759000,3753000,4649000,5937000

    ;表示21个年公司总收入的21个dword型数据

    dw 3,7,9,13,28,38,130,220,476,778,1001,1442,2258,2793,4037,5635,8226
    dw 11542,14430,15257,17800
    ; 以上是表示21年公司雇员人数的21个word型数据
data ends

table segment
;               0123456789abcdef
    db 21 dup ('year summ ne ?? ')
table ends

string segment
        db 10 dup ('0'),0
string ends

stack segment
        db 128 dup (0)
stack ends

codesg segment
start:  mov ax,stack
        mov ss,ax
        mov sp,128

        call init_reg

        call clear_screen

        call input_table
        call output_table



        mov ax,4c00H
        int 21H

;========================================
output_table:
        call init_reg_output_table
        mov si,0
        mov di,160*3

        mov cx,21

outputTable:
        call show_year  ;年份
        call show_income    ;收入
        call show_employee  ;人数
        call show_average_income    ;人均收入
        add di,160
        add si,16
        loop outputTable

        ret
;               0123456789abcdef
;    db 21 dup ('year summ ne ?? ')
;========================================
;========================================
show_average_income:
        push ax
        push dx
        push ds
        push si
        push di
        mov ax,ds:[si+13]
        mov dx,0

        mov si,9
        add di,30*2

        call show_number
        pop di
        pop si
        pop ds
        pop dx
        pop ax
        ret
;========================================
show_employee:
        push ax
        push dx
        push ds
        push si
        push di
        mov ax,ds:[si+10]
        mov dx,0

        mov si,9
        add di,20*2

        call show_number
        pop di
        pop si
        pop ds
        pop dx
        pop ax

        ret
;========================================
show_income:
        push ax
        push dx
        push ds
        push si
        push di
        mov ax,ds:[si+5]
        mov dx,ds:[si+7]

        mov si,9
        add di,10*2

        call show_number
        pop di
        pop si
        pop ds
        pop dx
        pop ax
        ret
;========================================

show_number:

        push ax
        push bx
        push cx
        push dx
        push ds
        push es
        push si
        push di
        push bp
        
        call isShortDiv
        ; 判断采取 什么样的除法
        ;长除法最终会变成 短除法

        call init_reg_show_string

        call show_string

        pop bp
        pop di
        pop si
        pop es
        pop ds
        pop dx
        pop cx
        pop bx
        pop ax

        ret

;========================================
show_string:
        push cx
        push ds
        push es
        push si
        push di
        mov cx,0

showString:
        mov cl,ds:[si]
        jcxz showStringRet
        mov es:[di],cl
        add di,2
        inc si
        jmp showString

showStringRet:
        pop di
        pop si
        pop es
        pop ds
        pop cx
        ret
;========================================
init_reg_show_string:
        mov bx,string
        mov ds,bx

        mov bx,0B800H
        mov es,bx
        ret
;========================================
isShortDiv:
        mov cx,dx
        jcxz short_div

        mov cx,10
        push ax
        mov bp,sp
        call long_div
        add sp,2
        add cl,30H
        mov es:[si],cl
        dec si
        jmp isShortDiv

shortDivRet:
        ret
;========================================
long_div:
        mov ax,dx
        mov dx,0
        div cx
        push ax
        mov ax,ss:[bp+0]
        div cx

        mov cx,dx
        pop dx
        ret
;========================================
short_div:
        mov cx,10
        div cx
        add dl,30H
        mov es:[si],dl
        mov cx,ax
        jcxz shortDivRet

        dec si
        mov dx,0
        jmp short_div

;========================================
show_year:
        push ax
        push bx
        push cx
        push ds
        push es
        push si
        push di
        mov bx,0B800H
        mov es,bx

        mov cx,4
        add di,3*2  ;设置显示年份的左间距
showYear:
        mov al,ds:[si]
        mov es:[di],al
        add di,2
        inc si
        loop showYear

        pop di
        pop si
        pop es
        pop ds
        pop cx
        pop bx 
        pop ax
        ret
;========================================
init_reg_output_table:
        mov bx,table
        mov ds,bx

        mov bx,string
        mov es,bx
        ret
;========================================
input_table:
        call init_reg_input_table

        mov si,0
        mov di,0
        mov bx,21*4*2

        mov cx,21

inputTable:
        push ds:[si+0]
        pop es:[di+0]
        push ds:[si+2]
        pop es:[di+2]

        mov ax,ds:[si+21*4+0]
        mov dx,ds:[si+21*4+2]
        mov es:[di+5],ax
        mov es:[di+7],dx

        push ds:[bx]
        pop es:[di+10]

        div word ptr ds:[bx]
        mov es:[di+13],ax

        add si,4
        add di,16
        add bx,2

        loop inputTable

;               0123456789abcdef
;    db 21 dup ('year summ ne ?? ')
        ret

;========================================
init_reg_input_table:
        mov bx,data
        mov ds,bx

        mov bx,table
        mov es,bx
        ret
;========================================
clear_screen:
        mov bx,0
        mov dx,0700H
        mov cx,2000

clear:  mov es:[bx],dx
        add bx,2
        loop clear
        ret
;========================================
init_reg:
        mov bx,0B800H
        mov es,bx

        mov bx,data
        mov ds,bx
        ret

codesg ends
end start
```
