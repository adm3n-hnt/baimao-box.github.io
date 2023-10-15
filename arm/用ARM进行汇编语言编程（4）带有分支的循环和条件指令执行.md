# ARM 编程模拟器
ARM 编程模拟器网站地址：
```
https://cpulator.01xz.net/?sys=arm-de1soc
```
在arm里也有和高级语言一样的for和while循环，可以根据条件来判断是否执行
# 带有分支的循环
首先我们创建一个data标签，然后在里面写一个分支，存放一些数值，然后使这些存放的数值依次相加
```
.global _start
_start:
	
.data   //data段，用于存放数据
list:   //定义的变量名
	.word 1,2,3,4,5,6,7,8,9,10  //.word：告诉程序每一个值都是word类型，大小为32位，用于存放数据
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/c64dd45572664e698612c571061dc7fd.png)


然后我们要将list加载到内存里
```
.global _start
_start:
	ldr r0,=list  //ldr：将data段里list中的数据加载到r0寄存器中
	 
.data
list:
	.word 1,2,3,4,5,6,7,8,9,10
```

然后使用直接寻址，将r0寄存器里的值放到r1寄存器里
```
.global _start
_start:
	ldr r0,=list
	ldr r1,[r0]   //汇编程序寻找r0寄存器地址中的值，移动到r1寄存器里
	
.data
list:
	.word 1,2,3,4,5,6,7,8,9,10
```

然后我们可以使用加法，将第一个值添加到r2寄存器里
```
.global _start
_start:
	ldr r0,=list
	ldr r1,[r0]   
	add r2,r2,r1  //r2 = r2 + r1 
	
.data
list:
	.word 1,2,3,4,5,6,7,8,9,10
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/bb321267f94c45888f90787af9e6e7cb.png)

现在按f5执行程序，看看内存里是什么样子的

![在这里插入图片描述](https://img-blog.csdnimg.cn/4b5663e9b2cc4e009ec83808f2d8786e.png)

存放的都是list里的值，1-10，之后就都是aaaaaaaa，aaaaaaaa的意思是内存中空闲的位置，我们可以利用这个aaaaaaaa，使遍历列表的时候告诉程序这就是列表的末尾了

我们可以将aaaaaaaa存放到某个变量里，但是在arm里，存放的值只能是特定大小，大小为两个十六进制值，明显aaaaaaaa大于了两个十六进制，这里我们就要使用常量了

常量很简单，在程序顶端输入.equ就可以定义了，我们可以给他一个名称和值，这和高级语言中定义常量的方法相同

```
.global _start
.equ endlist,0xaaaaaaaa   //定义一个常量，名称为endlist，内容为0xaaaaaaaa
_start:
	ldr r0,=list
	ldr r1,[r0]   
	add r2,r2,r1
	
	
.data
list:
	.word 1,2,3,4,5,6,7,8,9,10
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/888b8e90d25249e7a63fa27d12211c8b.png)

然后可以使用ldr将其加载到内存中
```
.global _start
.equ endlist,0xaaaaaaaa
_start:
	ldr r0,=list
	ldr r3,=endlist   //加载到r3寄存器中，之后写循环时方便利用
	ldr r1,[r0]   
	add r2,r2,r1
	
.data
list:
	.word 1,2,3,4,5,6,7,8,9,10
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/b670cb3dce91419fba64ed64a36f410d.png)

之后我们就可以开始写循环了，首先创建一个loop标签，加载list里的值
```
.global _start
.equ endlist,0xaaaaaaaa
_start:
	ldr r0,=list
	ldr r3,=endlist
	ldr r1,[r0]   
	add r2,r2,r1
loop:
	ldr r1,[r0,#4]!   //r0寄存器里的值偏移4字节，移动到r1寄存器里，然后他会查看内存中的那个位置是否是空的，之后将值存放在这里

.data
list:
	.word 1,2,3,4,5,6,7,8,9,10
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/cb9a6f66fe01413fab51d56e687cca13.png)



然后我们比较两个r1,r3，如果他们相等，我们就离开循环
```
.global _start
.equ endlist,0xaaaaaaaa
_start:
	ldr r0,=list
	ldr r3,=endlist
	ldr r1,[r0]   
	add r2,r2,r1
loop:
	ldr r1,[r0,#4]!
	cmp r1,r3   //比较这两个寄存器
	beq exit   //小于等于则调用exit标签
	
.data
list:
	.word 1,2,3,4,5,6,7,8,9,10
```
然后创建一个exit标签，不用写内容，他的作用就是程序的出口点
```
.global _start
.equ endlist,0xaaaaaaaa
_start:
	ldr r0,=list
	ldr r3,=endlist
	ldr r1,[r0]   
	add r2,r2,r1
loop:
	ldr r1,[r0,#4]!
	cmp r1,r3
	beq exit
	
exit:  //exit标签

.data
list:
	.word 1,2,3,4,5,6,7,8,9,10
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0dbfdef7c04842b1a24ffa185139ff37.png)

如果我们还没达到list的末尾，我们可以添加值到r2寄存器里，然后再次循环直到到达末尾
```
.global _start
.equ endlist,0xaaaaaaaa
_start:
	ldr r0,=list
	ldr r3,=endlist
	ldr r1,[r0]   
	add r2,r2,r1
loop:
	ldr r1,[r0,#4]!
	cmp r1,r3
	beq exit
	add r2,r2,r1  // //r2 = r2 + r1 
	bal loop  //然后通过跳转指令建立一个无限循环，直到r1和r3相等，到达list末尾
exit:

.data
list:
	.word 1,2,3,4,5,6,7,8,9,10
```
这就是一个完整的循环，作用是遍历list，使里面的值依此相加
之后运行程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/d072507d3283402d83cbaa583521b762.png)

然后进入循环，在循环10次后退出程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/4d8c25418e354198b2e7877a5a139303.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/9cf393b4c8e644feb32a80c8de9863a3.png)

1到10相加就等于55

# 条件指令执行
条件指令可以根据运算结果更新的条件标志，来判断指令的条件码是否符合条件，符合条件就执行，不符合条件则不执行

现在我们随便在寄存器里放一些值，然后比较
```
.global _start
_start:
	mov r0,#2
	mov r1,#4
	cmp r0,r1
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c459c77ec54249a5bc380c1e7189106f.png)

如果r0小于r1，就在r2寄存器里放入一个值
```
.global _start
_start:
	mov r0,#2
	mov r1,#4
	cmp r0,r1
	blt addr2   //如果r0小于r1，就调用addr2标签

addr2:
	add r2,#1  //r2+1
```
然后设置一个退出程序的出口
```
.global _start
_start:
	mov r0,#2
	mov r1,#4
	cmp r0,r1
	blt addr2
	bal exit  //跳转到exit标签里

addr2:
	add r2,#1
	
exit:
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/375634c6a03f49ab802f91f62f7d84e3.png)

仅仅实现这么简单的一个功能，需要的代码也太多了，有没有其他方法呢，我们可以使用条件指令来少写一些代码

还是随便在寄存器里放一些值然后比较
```
.global _start
_start:
	mov r0,#2
	mov r1,#4
	cmp r0,r1
```

之后可以使用addlt指令
```
addlt：意思是加小于(+<)，这个指令只会触发比较结果小于的情况
```
```
.global _start
_start:
	mov r0,#2
	mov r1,#4
	cmp r0,r1
	addlt r2,#1
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/e3895bd29e6141e1b72f8140300f32fc.png)

只用四行代码就实现了上面的功能，我们执行看看

![在这里插入图片描述](https://img-blog.csdnimg.cn/db9313a33e124f63ba616f580989ba4b.png)

可以看到，确实实现了我们想要的功能

比如还有movge……很多组合，这样可以使程序更加简洁
```
movge：比较结果是大于等于之后进行移动操作
```
# 总结
这是我学习的笔记，有什么错误和不懂的地方欢迎来私信我，或者加我qq
