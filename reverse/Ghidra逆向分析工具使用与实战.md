# 简介
Ghidra 是由美国国家安全局研究局创建和维护的软件逆向工程 (SRE) 框架 。该框架包括一套功能齐全的高端软件分析工具，使用户能够在包括 Windows、macOS 和 Linux 在内的各种平台上分析编译代码。功能包括反汇编、汇编、反编译、绘图和脚本，以及数百个其他功能。Ghidra 支持多种处理器指令集和可执行格式，并且可以在用户交互和自动化模式下运行。用户还可以使用 Java 或 Python 开发自己的 Ghidra 扩展组件和/或脚本。
下载地址：
```
https://github.com/NationalSecurityAgency/ghidra
```
# 使用
在github里下载后进入文件夹，安装java环境
```
apt install openjdk-11-jdk
```
然后运行工具
```
./ghidraRun
```
进入主界面
![在这里插入图片描述](https://img-blog.csdnimg.cn/8e7fac3de43e481db37251fd9c0b0f51.png)
新建一个项目或者打开一个项目
![在这里插入图片描述](https://img-blog.csdnimg.cn/6aa5c49ad0b44c48b534273e38960d17.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/4eecd8b714224461abaa3eb797ae7cb8.png)
next
![在这里插入图片描述](https://img-blog.csdnimg.cn/275c1ec780b34853b8476c1e5a5ad29a.png)
设置项目的存放地址，和名字
![在这里插入图片描述](https://img-blog.csdnimg.cn/e19f018969d44b8083d390b165641e1a.png)0
![在这里插入图片描述](https://img-blog.csdnimg.cn/4f8e2cc4ed154aa281924cedb6e4714d.png)
导入你想反汇编的文件，双击文件，然后会让你选择分析的东西，这里我们默认即可
![在这里插入图片描述](https://img-blog.csdnimg.cn/40a0c26c1ca94ab5a8d8d9d4c8eff2c3.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/ffe570cad4e243bb80d52168c044de35.png)
# 实战
![在这里插入图片描述](https://img-blog.csdnimg.cn/866b310dfcae4b7593e4ea20f6a55cbe.png)
使用ghidra逆向程序
查看程序中的字符串
![在这里插入图片描述](https://img-blog.csdnimg.cn/e41b8bf611e446f7865252bc4eecdab6.png)
找到刚刚显示在终端里的字符串
![在这里插入图片描述](https://img-blog.csdnimg.cn/b017e19845c14bbfaef357e6fd276d74.png)
然后双击去到函数调用的地方
![在这里插入图片描述](https://img-blog.csdnimg.cn/aaaf4d322da54c34a09c2b68a6755646.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/0b5b6fd7f9d84ae19447c130118ebe4a.png)
点击反汇编函数
![在这里插入图片描述](https://img-blog.csdnimg.cn/bf1c9c6bb6534122b3f51abbc3daa390.png)
我们就能看到和ida差不多的反汇编的界面了
分析伪代码
![在这里插入图片描述](https://img-blog.csdnimg.cn/3888c9ccd8c549df9172b08bc17be4a8.png)
在第二十一行的位置和0x86187做了一个对比，正确的话就弹出flag，错误的话就退出
我们将0x86187转换成十进制即可，右击，选择转换成10进制
![在这里插入图片描述](https://img-blog.csdnimg.cn/8dba0501b09b4b4dbab0e04d164bdf07.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/893ad0400a694b40a4fd11c6db94c8c7.png)
对比的数字为549255
我们运行程序后将此数字输入看看
![在这里插入图片描述](https://img-blog.csdnimg.cn/116e1d35298c4637aa14bb2e2a4aba79.png)
# 总结
这个工具我在外网看见的，并且工具本身也是开源的，我就拿来用一下试试，然后记录一下操作，平时我用的是ida和x64dbg，已经用习惯了







