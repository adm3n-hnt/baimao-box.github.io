# ARM 编程模拟器
ARM 编程模拟器网站地址：
```
https://cpulator.01xz.net/?sys=arm-de1soc
```
# 逻辑移位
```
LSL：逻辑左移
LSR：逻辑右移
```
这里有一个二进制00001010，转换为十进制为10，现在要进行LSL逻辑左移
```
00001010 --- 00010100  //每一位都向前移一位
```
现在就变成了00010100，十进制为20，就相当于乘以二了，我们可以用逻辑左移的方式，对数值乘以二

现在我们要进行LSR逻辑右移，还是二进制00001010，转换为十进制为10
```
00001010 --- 00000101   //每一位都向后移一位
```
现在就变成了00000101，转换为十进制为5，就相当于对数值除以二了

实战：

```
.global _start
_start:
	mov r0,#10
	lsl r0,#1  //lsl指令后面是需要移动的寄存器，和移动的位数
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/bd96ef62e1134b8ba744cd5b05059fec.png)

本来r0寄存器里面的值是10，现在二进制整体向左移了一位，就变成了20

![在这里插入图片描述](https://img-blog.csdnimg.cn/fc1b697bbb204cb48f84377ee3b9b331.png)
实战：

```
.global _start
_start:
	mov r0,#10
	lsr r0,#1   //lsr指令后面是需要移动的寄存器，和移动的位数
```
本来r0寄存器里面的值是10，现在二进制整体向右移了一位，就变成了5

![在这里插入图片描述](https://img-blog.csdnimg.cn/51f4297a79224c6aacca95f95fb42ce6.png)


# 轮换
```
ROR：右移轮换
```
轮换和逻辑移位很相似，但移位的时候不会损失数值
```
LSR：00001001 --- 00000100  //最后的1被移除了
ROR：00001001 --- 10000100  //1去到最左边了
```
需要注意的是，只有ROR，没有ROL，只能向右移，不能向左移

```
.global _start
_start:
	mov r0,#100
	ror r0,#1  //ror指令后面是需要移动的寄存器，和移动的位数
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/d917c2f54e46462f9837c20d1b8f00c1.png)
# 条件和分支
在高级语言里有if判断，在arm里也有if判断的指令
```
cmp：对操作的两个寄存器相减，不同的话就为正数或负数，相同就为0
bgt：b的意思是分支，gt的意思是大于
bge：大于等于
blt：小于
ble：小于等于
beq：等于
bne：不等于
```
```
.global _start
_start:
	mov r0,#2
	mov r1,#1
	cmp r0,r1   //r0 - r1
	bgt baimao  //当r0大于r1时，执行baimao标签里的内容
	
baimao:   
	mov r2,#5
```
在这里判断是正确的，r0大于r1时，他会直接跳到指定的标签里执行内容

![在这里插入图片描述](https://img-blog.csdnimg.cn/9a3f01226b7e4bd7af3e342577846615.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/79d2aa7ed4bb4f73b38ede05d7f0999a.png)

以此类推

还有一个指令是可以不用判断，直接执行标签里的内容
```
bal：执行指定标签里的内容
```
```
.global _start
_start:
	mov r0,#2
	mov r1,#1
	bal baimao
	
baimao:
	mov r2,#5
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/9f765e5e23914cb0911a606fb911835b.png)
# 总结
这是我学习的笔记，有什么错误和不懂的地方欢迎来私信我，或者加我qq

