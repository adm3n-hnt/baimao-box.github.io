# ARM 编程模拟器
ARM 编程模拟器网站地址：
```
https://cpulator.01xz.net/?sys=arm-de1soc
```
# 算数指令
```
add：加
sub：减
mul：乘
```
现在我们来写一个简单的小程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/1f9b49eb8d704925bea0d70f50ad24ca.png)

```
.global _start
_start:
	mov r0,#3  //r0 = 3
	mov r1,#7  //r1 = 7
	add r2,r0,r1    //r2 = r0 + r1 
```
运行程序，进入汇编页面

![在这里插入图片描述](https://img-blog.csdnimg.cn/1590bc9d72314eccb4a8dc2e5291dfbb.png)

可以看到，寄存器里的值都是我们输入的值，0a是十六进制，十进制为10
减法和加法一样


![在这里插入图片描述](https://img-blog.csdnimg.cn/d698a909eae24c3886f28a5df5e1b0f9.png)

```
.global _start
_start:
	mov r0,#11  //r0 = 11
	mov r1,#1  //r1 = 1
	sub r2,r0,r1    //r2 = r0 - r1 
```
运行程序，进入汇编页面

![在这里插入图片描述](https://img-blog.csdnimg.cn/117abb727f5f427d880c0f5be955e9ab.png)

乘法也是类似的

```
.global _start
_start:
	mov r0,#5  //r0 = 5
	mov r1,#5  //r1 = 5
	mul r2,r0,r1    //r2 = r0 * r1 
```

运行程序，进入汇编页面

![在这里插入图片描述](https://img-blog.csdnimg.cn/a69a997be7114ec9b1ef3974b3d32b61.png)

在我们运算的时候，总会遇到负数，我们看看在arm里是怎么样的

```
.global _start
_start:
	mov r0,#1  //r0 = 1
	mov r1,#6  //r1 = 6
	sub r2,r0,r1    //r2 = r0 - r1 
```

运行程序，进入汇编页面

![在这里插入图片描述](https://img-blog.csdnimg.cn/b28988bc89da4705ad845f82cfe60089.png)

r2寄存器里的值为一个很大的数，fffffffb，将十六进制转换为十进制时可以发现为-5

![在这里插入图片描述](https://img-blog.csdnimg.cn/597c5328f8a741628c1105b26d5514f4.png)

# CPSR寄存器
解决负数的方法就是使用的cpsr寄存器

![在这里插入图片描述](https://img-blog.csdnimg.cn/7732576b44a943d2b63e5ed43c692888.png)

在这里有一些字母，NZCVI SVC，这些字母都是cpsr寄存器里的特定标志

```
N：负数或小于
Z：零
C：进位或借位扩展
V：溢出
```

在这里，N代表负数，我们还可以用一些特别的指令来操作cpsr寄存器

```
.global _start
_start:
	mov r0,#1
	mov r1,#6
	subs r2,r0,r1  //当你不知道结果是否为负数时，可以在后面加S，这样就可以减少程序的报错
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/b6baa6e59301442fa4e4dcbe0138d52b.png)

运行程序，进入汇编页面

![在这里插入图片描述](https://img-blog.csdnimg.cn/a8756abef8714e86886ac34bc74ccad1.png)

当我们相加的结果超过了寄存器能存放的大小该怎么办，这时候还是要用CPSR寄存器里的特定标志

```
.global _start
_start:
	mov r0,#0xFFFFFFFF   //-1
	mov r1,#1
	adds r2,r0,r1  //因为相加后的结果超过了32位，就导致溢出了，设置s，之后cpsr寄存器就会进位
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/095bd99584fe45718fa972be2f202fe9.png)
# 逻辑运算
逻辑运算包括and，or，xor
在汇编里的表示如下：
```
and = and
or = orr
xor = eor
```
and:
```
.global _start
_start:
	mov r0,#0xff
	mov r1,#22
	and r2,r0,r1
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/7d5795b8f3f14661a1f587c00ee37c71.png)


or则相反：
```
.global _start
_start:
	mov r0,#0xff
	mov r1,#22
	orr r2,r0,r1
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/47f68c7813664699866a48d3666a1157.png)


xor：
```
.global _start
_start:
	mov r0,#2
	mov r1,#3
	eor r2,r0,r1   //将r0与r1里的值进行异或运算，输出结果到r2寄存器里
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/d53d5ead7df0441fa1c24ac31b5efa2b.png)

还有一个特殊的指令是mvn，作用是对正数+1求反
```
.global _start
_start:
	mov r0,#2
	mvn r0,r0
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/50d1cf2744054c8bbf36f43eee4fb16f.png)

r0变成了fffffffd，转换成十进制就是-3，and指令还有一个用
```
.global _start
_start:
	mov r0,#0xff
	mvn r0,r0
	ands r0,r0,#0x000000ff
```
在执行了mov指令后，r0寄存器里的值为0xff

![在这里插入图片描述](https://img-blog.csdnimg.cn/4c1653caaf3142b9ba9874f7c63b761c.png)

执行了mvn指令后，r0寄存器里的值为0xffffff00

![在这里插入图片描述](https://img-blog.csdnimg.cn/d9742b6840f44763900b7746238d2161.png)

进行了and指令后，r0寄存器为0

![在这里插入图片描述](https://img-blog.csdnimg.cn/bc344e81ed9442ea8a6489a92cc5ce1d.png)

# 总结
这是我学习的笔记，有什么错误和不懂的地方欢迎来私信我，或者加我qq
