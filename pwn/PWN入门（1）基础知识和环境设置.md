# 简介
"pwn"这个词的源起以及它被广泛地普遍使用的原因，源自于魔兽争霸某段讯息上设计师打字时拼错而造成的，原先的字词应该是"own"这个字，因为 'p' 与 'o' 在标准英文键盘上的位置是相邻的，PWN 也是一个黑客语法的俚语词，是指攻破设备或者系统。发音类似"砰"，对黑客而言，这就是成功实施黑客攻击的声音，而在ctf比赛里，pwn是对二进制漏洞的利用
# 缓冲区溢出
当系统向缓冲区写入的数据多于它可以容纳的数据时，就会发生缓冲区溢出或缓冲区溢出，用更简单的话说就是在程序运行时，系统会为程序在内存里生成一个固定空间，如果超过了这个空间，就会造成缓冲区溢出，可以导致程序运行失败、系统宕机、重新启动等后果。更为严重的是，甚至可以取得系统特权，进而进行各种非法操作。

关于缓冲区更多的解释，可以看我之前这一篇文章(用简单易懂的话语来快速入门windows缓冲区溢出)
```
https://blog.csdn.net/qq_45894840/article/details/123373180?spm=1001.2014.3001.5502
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/66e2ea6118514c999fb31538c8d89e54.png)

从这个显示图里可以看到缓冲区溢出的过程，在进行缓冲区溢出攻击后，缓冲区（buffer）里的字符向上溢出到了返回函数，从而导致程序错误，我们还可以利用这个返回地址，来达到远程入侵的目的

# 环境安装
## gdb
gdb是一个linux的动态调试器，可以调试各种二进制文件

```
apt install gdb 
```

## pwndbg
 pwndbg是一个 GDB 插件，它可以减少使用 GDB 进行调试，重点关注低级软件开发人员、硬件黑客、逆向工程师和漏洞利用开发人员所需的功能
```
git clone https://github.com/pwndbg/pwndbg
cd pwndbg
./setup.sh
cd ..
mv pwndbg ~/pwndbg-src
echo "source ~/pwndbg-src/gdbinit.py" > ~/.gdbinit_pwndbg
```
## pwntools
pwntools是一个模块，可以让我们写脚本时更加方便
```
python3 -m pip install --upgrade pwntools
```
# 动态调试
下载这个github库，进入第一个目录
```
https://github.com/Crypto-Cat/CTF/tree/main/pwn/binary_exploitation_101
```

这里可以看到两个文件，第一个是编译后的文件，第二个是源文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/43a788a5be7e4db4a4c9b7e5af8e71ce.png)

我们用vim打开源文件看看
```
vim vuln.c
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/bdbbdb8612c947518f30f21b2b06a240.png)

这是一个非常简单的c语言程序

```
#include <stdio.h>
#include <string.h>

int main(void)
{
    char buffer[16];    //定义了一个变量，名为buffer，有16个字节的缓冲区

    printf("Give me data plz: \n");   //输出字符Give me data plz:
    gets(buffer);    //获取我们输入的字符，并存入到buffer变量里
    
    return 0;
}
```

乍一看是不是没什么问题，其实是函数的问题

## gets
在gets函数的官方文档里，有这么一句话：

![在这里插入图片描述](https://img-blog.csdnimg.cn/aabe33ff5ec745f4a93d4b323bb27d3d.png)

永远不要使用gets函数，因为如果事先不知道数据，gets将就无法判断读取多少个字符，因为gets将继续存储字符当超过缓冲区的末端，使用它是极其危险的，它会破坏计算机安全，请改用fgets。

用之前的图片来解释

![在这里插入图片描述](https://img-blog.csdnimg.cn/77d64d0c49274ef7a9e8b8ad3bd0d76c.png)

## 程序信息

我们输入超过16个字节，缓冲区（buffer）就会向上溢出，覆盖程序本来正常的内容，从而造成程序崩溃

然后我们查看一下编译后的文件信息
```
file vuln
```
这是一个32位的可执行程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/179f44875f4a492cb0203ed8be243226.png)

intel x86架构的

![在这里插入图片描述](https://img-blog.csdnimg.cn/8b4114c342e9401890db6ab0c1b7ea7c.png)

动态链接，意思是程序内的函数都是在链接库里调用的，而不是本身就自带的

![在这里插入图片描述](https://img-blog.csdnimg.cn/c93e19e43b074b06ad58249760733289.png)

链接库地址为/lib/ld-linux.so.2

![在这里插入图片描述](https://img-blog.csdnimg.cn/01adb34e1dd545fc8a0546cdd553d203.png)

没有开启stripped，意思是如果我们用gdb去调试这个程序，可以看到函数名称，可以更方便的分析程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/b156c531e0494e19a3790102d9be9983.png)

使用checksec工具可以查看程序更详细的信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/81e84154d14f40f897a8d173b10a4b4f.png)

从上到下依次是
```
32位程序
部分RELRO，基本上所有程序都默认的有这个
没有开启栈保护
未启用nx，nx：数据执行
没有pie，意思是程序的内存空间不会被随机化
有读，写，和执行的段，意思是我们可以在程序里写入shellcode
```

现在我们执行程序，输入16个以上的字符试试

![在这里插入图片描述](https://img-blog.csdnimg.cn/8605e3126f244902a4976ad21633e7d5.png)

可以看到，程序已经报错了，因为我们溢出后覆盖了程序原本的一些参数

## 使用gdb调试程序
现在我们使用gdb来动态调试程序
```
gdb vuln
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/dc083ed5e7f44f038c31f5e810bb0367.png)

然后我们查看程序里所有调用的函数

```
info functions
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/829667ecdb8942d7ac2d326c7ffd220c.png)


然后查看主函数main的汇编代码
```
disassemble main
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/d8284f61825c4e0ca627a8e51e528b4c.png)

然后在main函数开头下一个断点，方便我们调试，断点的意思是程序运行时，遇到断点处会停下来
```
b main
run   //运行程序
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/1bf549147d664cc58a442a5d6ccf6730.png)

第一段是显示了寄存器里的所有内容，寄存器是内存中非常靠近cpu的区域，因此cpu可以快速访问它们，但是在这些寄存器里面能存储的东西非常有限

![在这里插入图片描述](https://img-blog.csdnimg.cn/ca1395ec2984456db5efd9aab2af279f.png)

第二段是当前程序停下来的位置和程序的汇编代码地址

![在这里插入图片描述](https://img-blog.csdnimg.cn/3dde36828e934cad90e09ceabe6ce1f0.png)


第三段是查看当前堆栈里的内容

![在这里插入图片描述](https://img-blog.csdnimg.cn/9ead5c814a71467e8dda1e42fa106633.png)

输入next执行下一个汇编指令

![在这里插入图片描述](https://img-blog.csdnimg.cn/0527d3902f6444519f3589fb7e74be7d.png)

或者一直输入n来到fget函数执行的地址

![在这里插入图片描述](https://img-blog.csdnimg.cn/1bc05b16c8084de7bea3bee3589ebb70.png)

输入一大堆A看看

可以看到，寄存器里的值都被A覆盖了

![在这里插入图片描述](https://img-blog.csdnimg.cn/2b6c003277674c55a8711d96904f37cc.png)

但最重要的是EIP这个寄存器，他是程序的指针，指针就是寻找地址的，指到什么地址，就会运行该地址的参数，控制了这个指针，就能控制整个程序的运行

# 静态调试
在linux里，我们可以使用ghidra这个工具来静态调试程序
下载地址：
```
https://github.com/NationalSecurityAgency/ghidra
```
程序安装和基本使用可以看我之前写过的文章（Ghidra逆向分析工具使用与实战）
```
https://blog.csdn.net/qq_45894840/article/details/124556441?spm=1001.2014.3001.5502
```
看我那篇文章创建了工作组后导入这个程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/a488d28301364b5da09bfbe364722422.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/e07bc5a25f7547bd937462050543b482.png)

然后双击进入分析

![在这里插入图片描述](https://img-blog.csdnimg.cn/c93451455a5546d7afa616a9f74d0e60.png)

选择yes，对程序进行默认解析

![在这里插入图片描述](https://img-blog.csdnimg.cn/692bf97dc49d4cb6aa585314f18af3b1.png)



![在这里插入图片描述](https://img-blog.csdnimg.cn/9c05cb7eadd54585a3256cabaf73f69c.png)

在函数这个文件夹可以找到程序调用的所有函数

![在这里插入图片描述](https://img-blog.csdnimg.cn/e6e0559433304277b21dc8e70c197fee.png)

点击main，中间的是汇编界面，右边是伪代码界面，通过分析汇编函数，程序会自动生成伪代码来帮助我们理解程序，可以看到，这里的伪代码和程序的源文件代码很相似

![在这里插入图片描述](https://img-blog.csdnimg.cn/ed82484cf07a42e8b04ea1a753cb102d.png)

在程序的window窗口可以查看程序里所有的字符串

![在这里插入图片描述](https://img-blog.csdnimg.cn/346361364fd043599d2421bf4dc133c2.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/0211a01297ce4fdc9fba148aa1820243.png)

单击后点击字符串被调用的地址就可以立即跳转到调用地方

![在这里插入图片描述](https://img-blog.csdnimg.cn/8d4bde86df5f4b0b9636de95a9c76d9d.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/a62e069844ed4b7f8e5cc7f3499e0f38.png)
# 总结
有什么不会或者是错误的地方的可以私信我，看到必回
