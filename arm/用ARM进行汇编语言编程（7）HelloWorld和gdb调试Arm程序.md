# 简介
上一篇文章我们在linux里安装了arm环境，不知道怎么安装的可以去看上一篇文章

# HelloWorld
连接ssh后，然后创建一个文件夹来存放我们写的项目
```
mkdir HelloWorld
cd HelloWorld
```
然后用nano工具创建一个文件
```
nano HelloWorld.s
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/8a009bf180754bd7aadb34424403c968.png)

首先创建基本的标签，写入我们想输出的字符串
```
.global _start
_start:


.data
message:
        .asciz "hello world !\n"    //.asciz：ascii集，后面是输出的字符
len = .-message   //定义字符串的长度
```
然后就要写主程序了，第一件事是我们要把数据或者字符存放到哪里，第二件事是要输出什么，第三件事是我们输出字符的长度是多少
```
.global _start
_start:

        MOV R0,#1   //计算机标准输出
        LDR R1,=message   //告诉程序字符串位置在哪里
        LDR R2,=len   //告诉程序要输出的字符串长度
        MOV R7,#4   //当我们与操作系统交互时，r7是一个特殊的寄存器，4意思是输出
		SWI 0  //中断程序

		MOV R7,#1   //1终止此程序
		SWI 0   //中断程序

.data   
message:
        .asciz "hello world !\n"
len = .-message
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/a41dc0dbc947488aa027073832ba1a27.png)

ctrl+o保存，然后ctrl+x退出

![在这里插入图片描述](https://img-blog.csdnimg.cn/6b24cff4b7bb4af5b156866b5d2b0479.png)

然后编译程序
```
as HelloWorld.s -o helloworld.o
ld helloworld.o -o hellworld
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a7198dc64e5e49d78910f7766c33e0ba.png)

执行
```
./helloworld
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/6f41aeaf89234a37a1579c8380c5b177.png)

成功输出
# gdb调试Arm程序

gdb是linux动态调试工具
```
chmod 777 helloworld
gdb helloworld
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/11954f4eb6344f4390e3fa65f064464d.png)

然后对程序添加断点，让程序停在我们想要分析的地址上
```
b _start   //在_start处添加断点
disassemble _start  //查看_start的汇编代码
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/6c4b2428fa6440888343994fe71fd984.png)

输入run就能停在断点处，现在程序是正在运行中的

![在这里插入图片描述](https://img-blog.csdnimg.cn/c94febebe7af4acc9ec07f564c0bff57.png)

输入layout asm 可以获得一个方便观察的界面

![在这里插入图片描述](https://img-blog.csdnimg.cn/3650d11c9bad45849b497be96fe040c0.png)

可以看到，这里的汇编代码和我们写的差不多，输入info register [小写的寄存器] 可以查看当前寄存器的值，现在没有执行，所以是0

![在这里插入图片描述](https://img-blog.csdnimg.cn/ec9c4093239547ed98ebc5a8aae10b16.png)

或者输入layout regs，上面就会显示寄存器的界面

![在这里插入图片描述](https://img-blog.csdnimg.cn/6e5d2dbb828045f8b5ef0697181f9b93.png)

可以用键盘上的上下键查看其他地址，输入stepi，执行第一条汇编代码

![在这里插入图片描述](https://img-blog.csdnimg.cn/5f36cab315134f768d2e9fe18b4cf06a.png)

可以看到，r0寄存器里的值就为1了，使用x/16x $r1 查看r1内存地址里的值

![在这里插入图片描述](https://img-blog.csdnimg.cn/30dcbd7e29294b0ba6170ed8befc5144.png)

x/10d $r1以十进制显示内存里的数值

![在这里插入图片描述](https://img-blog.csdnimg.cn/1d1fc89ab175435bbac338d460d8c63c.png)

x/10c $r1以ascii码新式查看内存里的数据

![在这里插入图片描述](https://img-blog.csdnimg.cn/5359ea0f4b7d4e00adc7eb1d3a6c464e.png)

# 总结
这是我学习的笔记，有什么错误和不懂的地方欢迎来私信我，或者加我qq
