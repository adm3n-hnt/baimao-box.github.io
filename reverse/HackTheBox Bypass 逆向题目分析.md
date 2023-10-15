![在这里插入图片描述](https://img-blog.csdnimg.cn/e0c089402b354c7baf388bde4b47c92e.png)

题目地址：
https://app.hackthebox.com/challenges/114


# 开始

运行程序，随意输入一些值

![在这里插入图片描述](https://img-blog.csdnimg.cn/286fffb8a0804f6383eeb626adacf82f.png)


将程序拖入die分析

![在这里插入图片描述](https://img-blog.csdnimg.cn/3838aba3477e4918aaf61421ce7198c8.png)

这是一个32位的pe文件，代码库是用 .NET写的，这里我们使用dnSpy工具来动态调试

工具地址：
https://github.com/dnSpy/dnSpy


将程序拖入32位的dnSpy，找到main函数

![在这里插入图片描述](https://img-blog.csdnimg.cn/48cf007b7c8340fd97ab7074299522c3.png)

这里可以看到flag2 = flag，而且都是布尔值，这下就简单了，在这两个地方下一个断点，然后运行程序，将布尔值的false改成ture就好了

![在这里插入图片描述](https://img-blog.csdnimg.cn/4a4f7bade7354eb889ba3f6a60a2ac12.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/ef9d2245d1ef408d875db847cb65c081.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/e426ff34592b40c79ff8801b49a5ac9c.png)

这里叫我们输入key，我们随意输入试试

![在这里插入图片描述](https://img-blog.csdnimg.cn/64100efa03764182b63c47fe10ecb1eb.png)

在这里，程序将我们输入的值与一长串字符串做了对比，这里猜测就是密钥，我们直接输入看看

![在这里插入图片描述](https://img-blog.csdnimg.cn/6f2be155586e4dab8e9bb87b5ddfb9d9.png)

成功得到flag
