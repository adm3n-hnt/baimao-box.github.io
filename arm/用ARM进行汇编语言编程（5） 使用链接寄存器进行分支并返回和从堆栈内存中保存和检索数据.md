# ARM 编程模拟器
ARM 编程模拟器网站地址：
```
https://cpulator.01xz.net/?sys=arm-de1soc
```
在汇编里也有函数的概念，我们将使用函数看看如何调用一个位置并返回
```
调用：在程序里移动到不同的标签，然后执行标签里的内容
```
# 使用链接寄存器进行分支并返回
假设我要写一个程序，作用是将两个数值相加，然后将这个功能设置为一个函数，以便重复利用

首先我们随便移动一些值到寄存器里
```
.global _start
_start:
	mov r0,#1
	mov r1,#3
```
然后写一个分支，作用是相加两个寄存器的值
```
.global _start
_start:
	mov r0,#1
	mov r1,#3
	bal add2   //跳转到add2分支里执行内容

add2:
	add r2,r0,r1   //r2 = r0 + r1
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/7a8832b711b348508fd710f8cfec0435.png)

然后按f5执行程序，可以看到，这个程序以我们想要的方式执行了

![在这里插入图片描述](https://img-blog.csdnimg.cn/861c283cb5894667a04d9c4ce6082254.png)

但现在的问题是，如果我想在函数调用后继续执行主函数里的内容，但是在这里调用之后就退出程序了

我们可以添加一个返回地址就能回到主函数里了
```
.global _start
_start:
	mov r0,#1
	mov r1,#3
	bl add2   //bl：跳转指令，用于返回到跳转指令之后的那个指令继续执行
	mov r3,#3

add2:
	add r2,r0,r1
```
在使用了bl指令之后，在寄存器中有一个叫做lr的链接寄存器，作用是保存子程序返回地址，然后我们执行程序看看

![在这里插入图片描述](https://img-blog.csdnimg.cn/96f297b6959f4c19a5f1c051480aa5c0.png)


在实行了跳转之后，lr寄存器里的值就是之后mov操作的地址

之后可以使用bx指令
```
bx：跳转到指定寄存器里的位置
```
```
.global _start
_start:
	mov r0,#1
	mov r1,#3
	bl add2
	mov r3,#3

add2:
	add r2,r0,r1
	bx lr   //跳转到lr链接寄存器里的位置
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/159ba2c517f54146a64ed3556fa35324.png)

然后我们执行完整的程序看看

![在这里插入图片描述](https://img-blog.csdnimg.cn/e529290471724866b5df846dd613d4cb.png)

可以看到成功返回到了主函数里继续执行mov的操作了
# 从堆栈内存中保存和检索数据
在我们写程序时，为了进行不同的计算，我们需要在写的过程中使用不同的寄存器，但他内部的寄存器是有限的，该怎么办呢

我们可以在使用后恢复原来寄存器里的值
这里写一个简单的小程序来演示，首先移动一些值到寄存器里
```
.global _start
_start:
	mov r0,#1
	mov r1,#3
```
然后我们调用一个函数，移动一些值到相同的寄存器里，然后相加，最后返回
```
.global _start
_start:
	mov r0,#1
	mov r1,#3
	bl baimao   //跳转到baimao分支里执行内容
baimao:
	mov r0,#5
	mov r1,#5
	add r2,r0,r1   //r2 = r0 + r1
	bx lr   //跳转到lr链接寄存器里的位置
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0d6c9d694f0e46dcbc7105de2a33b7f1.png)

当我们进入baimao这个函数时，r0和r1寄存器里的值将被覆盖

![在这里插入图片描述](https://img-blog.csdnimg.cn/f76b403b39ba468d82bfe9fd524e92c6.png)

我们可以将r0和r1寄存器的值push进堆栈中，这样做的目的是，当程序执行完之后，我们还可以pop出堆栈，还原寄存器里的数值
```
.global _start
_start:
	mov r0,#1
	mov r1,#3
	push {r0,r1}   //将r0和r1寄存器里的值push进堆栈
	bl baimao
	pop {r0,r1}  //在函数执行完之后，将堆栈里的值弹出来，还原寄存器里的数值
	b end  //跳转到end分支，结束程序
baimao:
	mov r0,#5
	mov r1,#5
	add r2,r0,r1
	bx lr   //跳转到lr链接寄存器里的位置
end:
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/81c886cd12cc4cbd99fd2ed6e980d198.png)

然后我们运行程序，看看会发生什么

在执行了push操作后，可以在内存中发现1，3数值

![在这里插入图片描述](https://img-blog.csdnimg.cn/d757a759d5c5405bb296de584c1d42c5.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/856e744520e44b47bb20ed0c79dc2d17.png)

而sp寄存器指向了存储值的位置，之后pop的时候可以直接还原寄存器里的值
```
sp：堆栈指针寄存器
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/8e3efc212c944c41aa66eae6dc465b50.png)

继续执行，可以看到我们的数据被覆盖了

![在这里插入图片描述](https://img-blog.csdnimg.cn/0449e4ad315e4ffe997ba3fdd7a8fa02.png)

返回执行了pop之后，值就还原了

![在这里插入图片描述](https://img-blog.csdnimg.cn/b5415971413d4daeabf3e922c411c84f.png)

# 总结
这是我学习的笔记，有什么错误和不懂的地方欢迎来私信我，或者加我qq



