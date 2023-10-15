# 简介
为了深入学习二进制安全，而进一步的对linux系统方面的学习
在本篇中，将了解什么是系统调用，通过查看特定的内核函数 ==copy_from_user== 来理解用户模式和系统模式的含义
# 开始
我们先看看存在哪些类型的系统调用，在linux手册中可以阅读有关系统调用的信息
```
man syscalls
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/300156f1f08241f3a6ff09a6be40975f.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/99462575bc6449ca94f46553761eb7bc.png)
这里说“系统调用是应用程序和linux内核之间的基本接口”
![在这里插入图片描述](https://img-blog.csdnimg.cn/e004d985006943d7959731bf411be04e.png)
这里还说“系统调用通常不直接调用，而是通过glibc中的包装函数调用”
![在这里插入图片描述](https://img-blog.csdnimg.cn/999b96e0537d47119072218d2da231e3.png)
“通常gilbc包装函数非常小，除了在调用系统调用之前将参数复制到正确的寄存器之外几乎无法做任何工作”
再往下，我们发现了大量可用的系统调用
![在这里插入图片描述](https://img-blog.csdnimg.cn/2e509f3a536a43aebf6b0dc882f8732b.png)
而libc函数printf也只是系统调用写入的一个包装器
我们创建一个文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/51f8ff73274644b798df04a5830595d3.png)
然后使用gcc编译文件
```
gcc license_1.c -o license_1    #-o，输出文件名称
```
```
strace ./license_1
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/37116687f31c4300a9becc7c2a16b23b.png)
可以看到，当我们使用程序strace来跟踪所有的系统调用时，它没有显示printf，而是显示了write
现在我们打开write的手册看看
```
man 2 write
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/8c0d424c49774daeb49390e0d1c28bc8.png)
这里说write函数需要三个参数
```
fd：文件描述符
buf：缓冲区地址
count：计数
```
然后我们创建一个简单的c程序来调用这个函数
```
#include <unistd.h>
void main(){
	write(1,"baimao\n",10);
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/01dbacfc62804e3c9ca498bfd933685a.png)


第一个参数是文件描述符，我们将他设置为1，意思是标准的文件描述符，对于第二个函数，我们要指向字符串的内存地址，我们可以在这里随意写一个字符串，然后编译器会在内存中为它找到一个位置并将它的地址放在那，最后一个参数是字符串的长度，在这个例子中，我们给字符串10的长度，如果给的长度小于字符串的长度，那么字符串将无法显示完整
然后我们gcc编译c文件，但是要关闭一些防护，不然不方便调试
```
gcc -z execstack -fno-stack-protector -no-pie -z norelro test.c -o test
./test
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/09c1a7e112964ca294c15dd3323a5961.png)

然后使用radare2来详细的看函数是怎么调用的

安装方法
```
apt install radare2
```
```
radare2 -d ./test
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/140ef93dea8d4572a5ebb0b69c6729f8.png)
操作的时候我的环境出了点问题，于是在kali上继续分析文件
使用参数aaa对程序自动分析自动命名
```
aaa
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/1bdef9714d01455a88056e1c95a5af95.png)

查看找到所有的函数
```
afl
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/4267165a4a134677ac0f255649d18b19.png)

找到main函数的入口
```
s main
```
然后查看程序汇编代码
```
pdf
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c9f94f07f40843ee86ca4d328ca1594c.png)
在调用write函数的位置下一个断点
```
db 0x0040115a
```
![加粗样式](https://img-blog.csdnimg.cn/864bff9d19c640e0a7af489a6279dd89.png)
运行文件到断点处
```
dc
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/d9adff0b27dd432cbf462bd82e95076b.png)
输入V!切换到可视化模式
![在这里插入图片描述](https://img-blog.csdnimg.cn/7a9dfe96206a46f2aa381602f20fcb8e.png)
按s可以一步一步详细的看程序的调用
![在这里插入图片描述](https://img-blog.csdnimg.cn/778c104505f74324be19c062ad384f8b.png)
这是链接表plt，我们将跳转到这里
![在这里插入图片描述](https://img-blog.csdnimg.cn/daaef75b69ed47dca6e18e24fe302d36.png)
这些都是libc库的实际代码，继续往下，过了很久，终于找到了关键的地方
![在这里插入图片描述](https://img-blog.csdnimg.cn/6d07edd46ed344e5aec378a505f02b6d.png)

这里他将1移入eax寄存器，然后是执行syscall指令
英特尔汇编器官方解释是
![在这里插入图片描述](https://img-blog.csdnimg.cn/8dafb1ac2adf42b19f57f36f54539cf3.png)
”这是对特权级别0，系统的快速调用“，并且操作码为==0F 05==
![在这里插入图片描述](https://img-blog.csdnimg.cn/6828f7e186cc436181a2ec7b68f381b8.png)

它还说“是通过从IA32_LSTAR MSR加载rip来实现的”，MSR是特定于模型的寄存器，因此就像将rip设置为另一个值，方便在其他地方继续执行跳转一样，它从模型特定寄存器（MSR）加载rip

![在这里插入图片描述](https://img-blog.csdnimg.cn/d83186f580cd4a65a2b9ffbbb5d52050.png)
这里说，在通过WRMSR指令引导系统期间的某个时间点，此地址就已经在特殊寄存器中配置，但是要使用此指令，必须处于特权级别0，因此不能从简单的c程序中设置它，因为处在用户模式，用户模式处在特权级别3
![在这里插入图片描述](https://img-blog.csdnimg.cn/ccca25858cc946ec8c57e705866c86b0.png)
如果想知道如何从级别3进入级别0，那么答案是通过syscall之类的指令
![在这里插入图片描述](https://img-blog.csdnimg.cn/4b2e6721b1954c5392e32af3ba749bd0.png)
但是当你在级别0时，无法控制执行想要的操作，因为它会跳转到一个预定义的地址
当打开计算机时，cpu从级别0开始，内核可以通过WRMSR指令配置地址，列如上面提到的LA32——LSTAR MSR寄存器，然后将cpu的权限降为3级，这时，我们无法重新配置寄存器，也无法重新配置硬件，只能通过系统调用再次进入级别0，我们也无法控制执行的内容，因为该地址是固定的
回到syscall调用
![在这里插入图片描述](https://img-blog.csdnimg.cn/25aa7dd8c0f84a28993fe76601ca92eb.png)
这里在做的是从寄存器中加载一个数字，在我们的例子中是1，然后我们通过系统调用跳转到内核中固定的地址来进入特权级别0，内核从寄存器中获取数字，它知道是从哪个系统中调用的
这个网站解释了为什么write系统调用的编号为1
```
https://filippo.io/linux-syscall-table/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/26ce929e3ce441358bdaa24bfb9180ba.png)
它是在read_write.c源代码中实现的，所以内核知道它需要执行什么
![在这里插入图片描述](https://img-blog.csdnimg.cn/719b922977104333a2e4c920c433e249.png)
这是当在调用write系统调用时，内核执行的内容
这里推荐一本免费的书，它叫做linux设备驱动程序

https://lwn.net/Kernel/LDD3/

![在这里插入图片描述](https://img-blog.csdnimg.cn/3cda3e2fbbb14e2faaaeefb17bc3a5a7.png)
这本书详细的介绍了内核的工作原理，尤其是如何编写设备驱动程序和内核模块
在书里的第三章第7节中写道“scull中读写代码需要将整个数据段复制到用户地址空间或从用户地址空间复制”
![在这里插入图片描述](https://img-blog.csdnimg.cn/6a34929328414d07b26349b4d6611b88.png)
“此功能由以下内核函数提供，它们复制任意字节数组并位于大多数读写实现的核心”
![在这里插入图片描述](https://img-blog.csdnimg.cn/141fb3df226c447199d3dc54549b52c5.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b3710e1a2f96456c8bc4b0f689a0888a.png)
首先，用户地址空间是什么意思？当使用gdb调试某些程序时，为什么每个程序似乎都使用相同的地址？代码始终位于相同的地址，所有程序如何使用内存中相同的地址？它们不会相互覆盖吗？
这就是我们拥有MMU（内存管理单元）的原因
![在这里插入图片描述](https://img-blog.csdnimg.cn/f8923fd8d1c44c5c9ff510343bd5195b.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/303f1ecd8a29417d9a812a1893c1688b.png)
内核使用特殊的cpu指令和配置寄存器等设置MMU，这告诉了MMU如何在虚拟地址和物理地址之间进行转换
因此，当在c程序中使用指令
```
mov eax,[0x04000000]
```
MMU会知道如何将此地址转换为RAM中的实际物理地址

因此，当我在系统调用进入内核后，希望从用户地址空间复制一些数据，列如将其写入其他地方，这时，可以使用copy_from_user函数
我们使用Linux Cross Reference搜索内核函数源代码
```
https://elixir.bootlin.com/linux/latest/A/ident/_copy_from_user
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/85543387cd2646d0881d9e8c4dd2a217.png)
点击第一个
![在这里插入图片描述](https://img-blog.csdnimg.cn/926be9ed87354b589c28c626f1b5469f.png)

该函数在from参数上调用access_ok，这是用户指定的地址，在我们上面那个例子中是我们想要写入的字符串的地址
![在这里插入图片描述](https://img-blog.csdnimg.cn/7443373ad1c94a7aadc67573559d863b.png)

这里将要检查是否要允许此进程从地址中读取信息，如果进程试图从另一个进程中读取一些信息，一切正常的话，它会调用__copy_from_user，点击函数，进入新的查询页面

![在这里插入图片描述](https://img-blog.csdnimg.cn/5914e8b19b0848b9b1eea659d7ec599e.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/744095eecb514a0799ded6cfeb10906e.png)
选择linux版本为4.3，进入此源代码，x86架构，64位
![在这里插入图片描述](https://img-blog.csdnimg.cn/10e25fac0a624934ab06e8f3a5c38d2e.png)
这看起来只是对copy_from_user_nocheck的包装，表示后面的函数不会再检查访问权限
点击copy_from_user_nocheck函数，进入新的查找页面
![在这里插入图片描述](https://img-blog.csdnimg.cn/0839dddc4a88428d9e43394e6e31a9c6.png)

点击第一个，进入函数源代码
![在这里插入图片描述](https://img-blog.csdnimg.cn/268edf930ae34d23970f0f7f8cc562b0.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/3c093cf67ee345cc8ce7e608719b151f.png)
这里有一个switch-case语句，它似乎检查了我们想要从用户空间读取的大小
![在这里插入图片描述](https://img-blog.csdnimg.cn/714db095dfa140a0a449e4b5ffa9c907.png)
假设我们只想从用户空间读取一个字节，这种情况下将返回这些
![在这里插入图片描述](https://img-blog.csdnimg.cn/a54082b3da5943ea985894a059ead77b.png)
所以__get_user_asm是一个预处理器宏，点击__get_user_asm，进入新的查找页面，我们将要了解如何分阶段编译C文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/372ec10610d54bb9b562aa0c62279eea.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/7e10969fe04c4a7a97031a89792ad9fc.png)
该语句只是一个简单的复制和替换，所以这里定义的代码只是简单的复制到它之前使用的位置，然后编译器开始将它编译成机器代码，我们将这些代码复制到乌班图上看
```
#define __get_user_asm(x, addr, err, itype, rtype, ltype, errret)	\
	asm volatile(ASM_STAC "\n"					\
		     "1:	mov"itype" %2,%"rtype"1\n"		\
		     "2: " ASM_CLAC "\n"				\
		     ".section .fixup,\"ax\"\n"				\
		     "3:	mov %3,%0\n"				\
		     "	xor"itype" %"rtype"1,%"rtype"1\n"		\
		     "	jmp 2b\n"					\
		     ".previous\n"					\
		     _ASM_EXTABLE(1b, 3b)				\
		     : "=r" (err), ltype(x)				\
		     : "m" (__m(addr)), "i" (errret), "0" (err))
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/9f3d8f4e88ef4fb488478c63681c9a84.png)
输入
```
:%s/"itype"/b/g
:%s/"rtype"/b/g
:%s/\\//g
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/7efd537ed7b94068b2e92a4bcc24e425.png)
回车
![在这里插入图片描述](https://img-blog.csdnimg.cn/4b5a5475bc9c492aaed368470b1351d0.png)
get_user_asm定义了一些实际的cpu指令，而这里的这个移动就是将数据从用户空间移动到这里变量中的指令
![在这里插入图片描述](https://img-blog.csdnimg.cn/c628e65ff45e491a853ab20208fbc992.png)
并且将它们设置为单个字节的“b”，在这些预处理器语句的工作时，只需要将此文本替换为b，所以实际指令看起来像
```
movb %2,%b1
```
这是at&t汇编语法，这意味着它将%2中的任何内容移动到%b1中
![在这里插入图片描述](https://img-blog.csdnimg.cn/6af57e9b0a6845739f7122f2e7289d39.png)
这些是c内联的汇编语法，它指定了在此处定义的变量，所以上面的%2指的是这三个参数，这些是我们要从中移动的数据地址，把它移动到%1中，也就是x，这就是我们将其移动到的位置
![在这里插入图片描述](https://img-blog.csdnimg.cn/f23d7fe2306b4ad5ab9b6f5547d46aad.png)

移动由STAC和CLAC操作，代表设置和清除寄存器，回到网站，点击ASM_STAC函数查看源代码，点击第一个
![在这里插入图片描述](https://img-blog.csdnimg.cn/eea954d3f7b34df1a80c6824c9807146.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/5052d0113d80467c9b2d30809f3f7555.png)

它与SMAP（一种反利用功能）有关，还有来自该指令的原始操作码
回到刚刚的__get_user_asm源代码
![在这里插入图片描述](https://img-blog.csdnimg.cn/541fefd73cba4ab5958294f4d589520a.png)

在mov下方，我们看到了部分修复和汇编异常表，这些涉及到内核如何处理硬件异常，这里有一个文档，可以在这上面阅读到它在那里的确切作用
```
https://www.kernel.org/doc/Documentation/x86/exception-tables.txt
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/221e4fd312f747cdb67b4f312a0af20a.png)
无论如何，没有代码可以以某种方式将用户提供的虚拟地址转换为真实的物理地址
![在这里插入图片描述](https://img-blog.csdnimg.cn/9460d7cdb973449da8b628c81d02b248.png)
它只是执行一个mov，因为这些操作发生在别处，那是因为这些操作发生在别处，当内核执行到这条指令时，会导致页缺失，因为它试图访问一个虚拟地址
![加粗样式](https://img-blog.csdnimg.cn/27392166c31d48c58b6b144c74952e61.png)
这会造成中断，cpu跳转到内核中另一个预定义的代码位置，该位置会处理异常，与系统调用指令让我们跳转到预定义地址的方式非常相似# 总结
# 总结
写了好几天，点个赞吧
