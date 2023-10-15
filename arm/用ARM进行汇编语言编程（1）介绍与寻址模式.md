# 简介
汇编语言是计算机的低级编程语言，它是最接近机器语言的语言，arm是一种用于各种不同应用程序的编程语言，通过arm汇编语言，程序员可以在较低的层次上编程与硬件交互的代码

# ARM 编程模拟器
ARM 编程模拟器网站地址：
```
https://cpulator.01xz.net/?sys=arm-de1soc
```
进入这个网站

![在这里插入图片描述](https://img-blog.csdnimg.cn/921fff241f6445b08354de0f47102b2d.png)

这个地方是显示寄存器的位置

![在这里插入图片描述](https://img-blog.csdnimg.cn/274d3a6124fd42e49f6991a4c696450a.png)

寄存器是内存中非常靠近cpu的区域，因此可以快速访问它们，但是在这些寄存器里面能存储的东西非常有限
这个地方显示内存里的值

![在这里插入图片描述](https://img-blog.csdnimg.cn/5742ffb8eab5430e9f6a951af163f58f.png)

sp寄存器会告诉我们堆栈上下一个可以用内存的地址，现在sp里的值全是0，所以我们下一个可用的内存地址的值在这里

![在这里插入图片描述](https://img-blog.csdnimg.cn/aebe4824ac4d48cb8acc6c96bbae53a9.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/607a4f764db44cf99d71489044ccdeef.png)

现在我选中的地址为0，在它旁边的地址为4，以此类推

![在这里插入图片描述](https://img-blog.csdnimg.cn/e0f755a7f58047f4b2d995f861fff0ed.png)

现在选中的这个位置为16，以此类推

![在这里插入图片描述](https://img-blog.csdnimg.cn/e2ddb3f43bc74d59a83526e42bad4464.png)

LR寄存器是我们的链接寄存器，如果使用过高级语言就会知道，当你有一个函数时，函数就会有一个返回值，而这个值会让我们回到调用函数的位置，这就是链接寄存器的作用

![在这里插入图片描述](https://img-blog.csdnimg.cn/3d0a6dab7d4b44eb9b746b503d9c03c2.png)

PC是我们的程序计数器，它的作用是跟踪要执行的下一条指令的位置

![在这里插入图片描述](https://img-blog.csdnimg.cn/d8648085100944b3896fea3f12bfbef6.png)
# 第一个程序
在我们程序的起点，模拟器会自动放入两行开始代码，告诉程序的起点在哪，如果在模拟器之外的地方编程，需要自己手动写入

![在这里插入图片描述](https://img-blog.csdnimg.cn/bf075e7f44b04101a7d3d2ab66adea82.png)

我们将在_start:下面写入代码，_start:被称为标签，标签和高级语言中函数差不多，这是一种能划分代码段的方法
我们先来写一个简单的代码，代码的作用是将一些数据移动到寄存器里，然后在将一些数据移动到另一个寄存器里，我们看看数据是如何移动到寄存器里的
```
mov r0,#30                   //将30移动到寄存器0中
mov r7,#1                    //将1移动到寄存器7中
SWI 0                        //中断程序
```
然后按下f5运行代码

![在这里插入图片描述](https://img-blog.csdnimg.cn/07d36fccd19d4324b7e287f9f38bc4cc.png)

现在这个界面为反汇编界面，然后按下f2，一步一步执行代码

![在这里插入图片描述](https://img-blog.csdnimg.cn/36647a2f025349aca5fbdbba37851548.png)

可以看到，r0寄存器里的值已经改变了，1e与十进制的30相同，
在这里可以将寄存器十六进制的值转换为十进制

![在这里插入图片描述](https://img-blog.csdnimg.cn/bac0583fbcc3425f9120630fcbd9496f.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/d54f4b9e38cf4aa091b623e8fc5e45f2.png)

# 寻址类型
寻址类型的定义是在程序的内存位置上可以通过多种方式从内存中存储和检索数据，在上面我们就运用了一种寻址类型，叫做立即寻址
```
mov r0,#30                   //将30移动到寄存器0中
mov r7,#1                    //将1移动到寄存器7中
```
取一个值，将其直接放入一个寄存器，这种类型就叫做立即寻址
我们还可以将一个寄存器里的值，放入另一个寄存器中
```
mov r1,#1     //将1放入寄存器1中
mov r2,r1    //将r1寄存器里的值放入r2寄存器中
```

我们还可以定义一个数据段在里面存放数据，之后需要调用时可以直接调用这个数据段
```
.data    //data段，用于存放数据
```
之后我们可以随意在里面定义变量

![在这里插入图片描述](https://img-blog.csdnimg.cn/15b2951a278b4a8eaa6f141c84a37495.png)

```
.global _start
_start:
	
.data
baimao:   //定义的变量名
	.word 1,2,-3,4,5,-6,7,8,-9,10   //.word：告诉程序每一个值都是word类型，大小为32位，用于存放数据
```
现在我需要调用这个变量在内存中的位置，只需要在baimao里查找第一个条目就好了，它会从第一个开始依次出现在内存中
```
.global _start
_start:
	ldr r0,=baimao     //ldr：将data段里baimao中的数据加载到r0寄存器中
.data
baimao:   //定义的变量名
	.word 1,2,-3,4,5,-6,7,8,-9,10   //.word：告诉程序每一个值都是word类型，大小为32位，用于存放数据
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/e0bc23d98d0749acadbf83afc323a70c.png)


然后按f5运行，进入汇编页面

![在这里插入图片描述](https://img-blog.csdnimg.cn/e5f4e6ad92c54df6a553fc14420ddfd3.png)

然后在右下角设置位十进制显示

![在这里插入图片描述](https://img-blog.csdnimg.cn/e44f688faae94a5187ea86de7fb3be36.png)

ctrl+m进入内存页面，就能以十进制显示我们的数据

![在这里插入图片描述](https://img-blog.csdnimg.cn/787280cab98e47b8af0053af3f86ad16.png)



我们还可以用另一个寻址模式来演示一下从该位置检索值
```
.global _start
_start:
	ldr r0,=baimao     //ldr：将data段里baimao中的数据加载到r0寄存器中
	ldr r1,[r0]   //汇编程序寻找r0寄存器地址中的值，移动到r1寄存器里
.data
baimao:   //定义的变量名
	.word 1,2,-3,4,5,-6,7,8,-9,10   //.word：告诉程序每一个值都是word类型，大小为32位，用于存放数据
```

按f5汇编后，进入汇编页面

![在这里插入图片描述](https://img-blog.csdnimg.cn/61fffea0ff4c4d4fac50344ca2e1d74c.png)

按f2执行程序后可以看到，r0寄存器里的值为16

![在这里插入图片描述](https://img-blog.csdnimg.cn/959aa611d8c845d981542fac199f26be.png)

进入内存，可以发现确实存在一个16的值，后面的那个0为方括号的值

![在这里插入图片描述](https://img-blog.csdnimg.cn/63d55b9c8b0e4f578190122cca19b62d.png)

继续按下f2后，r1寄存器里的值就是我们之前定义的值了

![在这里插入图片描述](https://img-blog.csdnimg.cn/8a799f9dd3564c87986a1972a51b7493.png)

r0寄存器里的值就是告诉程序从哪个内存地址里获取值，这里是16，所以在内存里也看到了16，紧挨着的就是我们定义的一些值


这里我用python来演示一下，可以更好的理解
```
.global _start
_start:
	ldr r0,=baimao     //ldr：将data段里baimao中的数据加载到r0寄存器中
	ldr r1,[r0]   //汇编程序寻找r0寄存器地址中的值，移动到r1寄存器里
.data
baimao:   //定义的变量名
	.word 1,2,-3,4,5,-6,7,8,-9,10   //.word：告诉程序每一个值都是word类型，大小为32位，用于存放数据
```
```
//上面那个汇编程序

baimao = [1,2,-3,4,5,-6,7,8,-9,10]
baimao[i]    //i就相当于r0寄存器的作用
```

还有其他方法可以调用变量里的值，比如说，我只需要2以后的这些值，该怎么办呢
这里我们可以使用偏移量来达到我们想要的值

```
.global _start
_start:
	ldr r0,=baimao     //ldr：将data段里baimao中的数据加载到r0寄存器中
	ldr r1,[r0]   //汇编程序寻找r0寄存器地址中的值，移动到r1寄存器里
	ldr r2,[r0,#4]   //偏移4字节，移动到r1寄存器里
.data
baimao:   //定义的变量名
	.word 1,2,-3,4,5,-6,7,8,-9,10   //.word：告诉程序每一个值都是word类型，大小为32位，用于存放数据
```
这里我用python来演示一下，可以更好的理解

```
//上面那个汇编程序

baimao = [1,2,-3,4,5,-6,7,8,-9,10]
baimao[i+1]    //+1就相当于偏移量
```

因为在内存中下一个值的位置是四个字节，所以这里偏移4位

![在这里插入图片描述](https://img-blog.csdnimg.cn/8d74f3e9987049bd851720c4d8c05d44.png)

运行程序，可以看到r2寄存器里的值变成了2

![在这里插入图片描述](https://img-blog.csdnimg.cn/fe70ff952b634101a8ce1dbb50760faf.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2f47dcdb089041b8b4dfe60ece576ec3.png)

# 总结
这是我学习的笔记，有什么错误和不懂的地方欢迎来私信我，或者加我qq


