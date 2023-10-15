![在这里插入图片描述](https://img-blog.csdnimg.cn/32147c50bbeb469fbc9f859a717f392a.png)
题目链接：
```
https://app.hackthebox.com/challenges/26
```
# 开始
die分析程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/c3950bdea1a14d0bab77a4bd5d524256.png)

64位的程序，其他就没有什么特别之处了，但是用ida无法打开


![在这里插入图片描述](https://img-blog.csdnimg.cn/5053c1dc38534ad0aea8ae650bd8a122.png)

用strings查看程序内的字符串

![在这里插入图片描述](https://img-blog.csdnimg.cn/152eb4d215194a6cb763b6fd1fcda86a.png)

发现一个英语单词
```
SuperSeKretKey
```
我们运行程序输入这个单词试试

![在这里插入图片描述](https://img-blog.csdnimg.cn/7b94a48e3f4d4684a8ce6663c8848a0f.png)

这个程序要经过两次验证才能输出flag，使用ltrace跟踪一下程序试试

![在这里插入图片描述](https://img-blog.csdnimg.cn/2b833feb77d74cf79820328b65d8aadc.png)

可以看到，我们输入SuperSeKretKey后，和内置的字符串对比了一下，然后我们随便输入一个字符串试试

![在这里插入图片描述](https://img-blog.csdnimg.cn/e352d20628da4561893e954f72ca7c4f.png)

可以看到，这个程序调用了time函数，参数为0，说明调用了当前的时间作为参数来随机生成一个字符串和我们输入的值比较，这里我们需要动态调试来破解程序

这里先使用ghidra来查看一下程序的伪代码

![在这里插入图片描述](https://img-blog.csdnimg.cn/3b4150103ea648e4846fbf7be44baf84.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/1ab630badbf94b1583dff12085248c67.png)

可以看到，程序的伪代码和我们分析的差不多，只需要绕过验证即可，接下来使用r2来动态破解程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/0b9cf07a537d466483fbf4802cfbf9b2.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/e282249ae64d4d6a9f19aacc62d600bf.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/1956499a1e01495389f74ef368031bb0.png)

在第一次 strcmp() 调用之后，它将我们的输入与该字符串进行比较

![在这里插入图片描述](https://img-blog.csdnimg.cn/eb7c0965261e4eb699860f34f41452fb.png)

如果 strcmp 返回 0，将跳转到0x400925，否则指令指针将执行下一条指令，将 1 移动到堆栈然后调用 exit

在下面还找了一个fcn.0040078d的函数，我们进去看看


![在这里插入图片描述](https://img-blog.csdnimg.cn/eaee5f35843248ffbbb8d86cbfd60675.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/002ca2a149d24025be6281dc9cc6397a.png)

在 0x004007b3 上调用 time(0)，在 0x004007d9 上调用 srand()，在 0x004007e9 上调用 malloc()，然后进行了一个循环，和我们之前的猜想一样，调用了当前的时间作为参数来随机生成一个字符串和我们输入的值比较

回到main函数，在此次还进行了一次对比

![在这里插入图片描述](https://img-blog.csdnimg.cn/b1911ffeb2d744ddaaf85193fa064d9b.png)
我看完整个程序的汇编代码后，找到了三种解决方案

第一个是nop掉我们输入错误后执行jmp指令的地址
第二个是在test指令执行后修改eax的值为1，这样不管我们输入正确与否，都会判断为正确
第三个是直接跳转到直接调用flag的地址

这里我演示第一种方法，直接nop掉指令

输入oo+，以读写的方式打开，然后跳转到这个地址

![在这里插入图片描述](https://img-blog.csdnimg.cn/c3363470ac594b48a2ad4467cc4e26b0.png)
```
s 0x00400968
```

然后修改程序，nop掉这个跳转操作
```
wx 9090
wa nop
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/9870b824351c4f949a512db323835a56.png)

然后我们在看看main函数

![在这里插入图片描述](https://img-blog.csdnimg.cn/3e802ecd721549339d0ab1440215f678.png)

已经成功修改了程序，现在我们输入q退出程序，然后运行

![在这里插入图片描述](https://img-blog.csdnimg.cn/a74aaeba5ac54d66b02078f2e494fcbd.png)

成功破解程序
