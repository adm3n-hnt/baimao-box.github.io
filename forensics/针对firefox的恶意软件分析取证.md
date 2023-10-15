# 简介
需要此文章用到的虚拟机环境和工具可以加我qq3316735898
# 环境介绍
环境用的是火眼Windows攻击工具集环境
该公司发布了一个包含超过140个开源Windows工具的大礼包，红队渗透测试员和蓝队防御人员均拥有了顶级侦察与漏洞利用程序集。该工具集名为“曼迪安特完全攻击虚拟机(CommandoVM)”，为安全研究人员执行攻击操作准备了即时可用的Windows环境
![在这里插入图片描述](https://img-blog.csdnimg.cn/7ef8a63c28014dc1b66a2b59cabb0e23.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/bf78164776b04907985ffe1f79a7a2b0.png)
此次分析的恶意软件为
![在这里插入图片描述](https://img-blog.csdnimg.cn/ee0396e80f2943a291d3bdc0e048b065.png)
# 恶意软件详细信息获取
首先用第三方网站对恶意程序进行分析，这里我使用VirusTotal，VirusTotal.com是一个免费的病毒、蠕虫、木马和各种恶意软件分析服务，可以针对可疑文件和网址进行快速检测，最初由Hispasec维护。它与传统杀毒软件的不同之处是它通过多种防毒引擎扫描文件。使用多种反病毒引擎可以令使用者们通过各防毒引擎的侦测结果，判断上传的档案是否为恶意软件
```
https://www.virustotal.com/gui/home/upload
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/492aa0ae512c4f0ebcc9e05012f2309a.png)
上传恶意软件进行分析
![在这里插入图片描述](https://img-blog.csdnimg.cn/a677bddf3be54122b04832dd0914849b.png)
然后去到文件详细信息选项，这里可以看到很多关于恶意软件的一些信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/869f33e948374b72a677e3b87fdc9551.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/fa4bdad1dd72475bbae086be54dacf49.png)
在这里可以知道binstall是一个浏览器助手的安装程序
![在这里插入图片描述](https://img-blog.csdnimg.cn/a30a08e8fb99461daaca284d1afffc11.png)
是一个.net的文件，也许是用C#写的程序
![在这里插入图片描述](https://img-blog.csdnimg.cn/c64a9f4273b548549ee9dd36683bf151.png)
我们还可以在其他分析平台上看看其他的详细信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/765025123f0347d1ad144f16f52c852b.png)
在这里，Joe Sandbox Analysis也对文件进行过分析，他可以运行文件并记录此文件做过的事情，我们可以点击下面链接去看看详细信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/c133c2c2413942beb84ac3c1f0bfaf60.png)
```
https://www.joesandbox.com/analysis/74132/0/html
```
Joe Sandbox Analysis将此文件标记为恶意
![在这里插入图片描述](https://img-blog.csdnimg.cn/63e69ee3d7c944638f22eeeb00087475.png)
在下面，有运行文件并记录过程的显示
![在这里插入图片描述](https://img-blog.csdnimg.cn/f23333e6fb004b3d9b5e61a1fb9b7442.png)
可以看到，这个文件创建了一个未记录的自启动注册表项，点击最右边的Show sources可以查看详情
![在这里插入图片描述](https://img-blog.csdnimg.cn/7781fde8b12a48b288d2437b451908b6.png)
还创建了一个pe文件，这个被创建文件名为browserassist.dll，pe文件是一个常规的windows二进制文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/980fb3e05b4a45dd906751e04af689b1.png)
# 恶意软件分析取证
回到虚拟机，我们使用peid对恶意软件进行分析，PEiD是一款著名的查壳工具，其功能强大，几乎可以侦测出所有的壳，其数量已超过470种PE文档的加壳类型和签名，可以探测大多数PE文件封包器、加密器和编译器
将文件拖入peid
![在这里插入图片描述](https://img-blog.csdnimg.cn/7364babbb95b46198157580942480d60.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/a49110f048db429fb1af1d34f9f20141.png)
可以看到，这是一个C#写的.net文件，
![在这里插入图片描述](https://img-blog.csdnimg.cn/1431c18954be407ebdce8e8ecdf77b3a.png)
从额外的信息猜测，有一部分内容可能是加密的
因为是C#写的.net文件，我们用ILSpy分析，ILspy是一个开源的.net反编译软件
位置在：
```
C:\ProgramData\chocolatey\lib\ilspy\tools
```
将恶意软件拖入进行分析
![在这里插入图片描述](https://img-blog.csdnimg.cn/d6ba7c555b304f7b87d1781f46a71371.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/45b6a79efe034ddbafeb1b96d6a9616e.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/f587277989404b67b8e44848953f9b78.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/6fbbcdf07edd4444be387993ad76f7f7.png)

可以看到，这些代码都被混淆了，但是他程序本身的一些功能似乎去实现解密的
![在这里插入图片描述](https://img-blog.csdnimg.cn/335335a7d1ce456a8f79755a582d4689.png)
这些都不重要，目前我们需要知道这个程序为什么要创建一个dll文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/b270d16c1d43424bad9bea7647d5edcc.png)
我们使用procmon对程序进行跟踪，Process Monitor一款系统进程监视软件，总体来说，Process Monitor相当于Filemon+Regmon，其中的Filemon专门用来监视系统 中的任何文件操作过程，而Regmon用来监视注册表的读写操作过程。 有了Process Monitor，使用者就可以对系统中的任何文件和 注册表操作同时进行监视和记录，通过注册表和文件读写的变化， 对于帮助诊断系统故障或是发现恶意软件、病毒或木马来说，非常有用
位置在：
```
C:\Tools\Sysinternals
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/3e55f9918d954e7ca7e63b15e7d47789.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/93eb15e0d0884886ac76445d6a334bed.png)
然后再打开process hacker，Process Hacker是一款针对高级用户的安全分析工具，它可以帮助研究人员检测和解决软件或进程在特定操作系统环境下遇到的问题。除此之外，它还可以检测恶意进程，并告知我们这些恶意进程想要实现的功能
位置在
```
C:\Program Files\Process Hacker 2
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/1bd61abd95a74975987aa379d40c592d.png)
打开程序
![在这里插入图片描述](https://img-blog.csdnimg.cn/51a75befceae460895a74c5f8800f509.png)
双击运行我们的恶意软件
![在这里插入图片描述](https://img-blog.csdnimg.cn/5d09d5bb576e4bd48be0d22477aca5bf.png)
回到Process Monitor，ctrl+f搜索恶意软件名字，我们就能看到这个程序进行了哪些操作
![在这里插入图片描述](https://img-blog.csdnimg.cn/96da940e9adb4ba29f556754e87261ae.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/80218ba34c47457abd4f15c59dccefc3.png)
再去挖掘一下被创建的dll文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/e7d0d2c89d7e4793b552e594857d9f0a.png)
打开api-monitor，勾选一些选项，进行更详细的分析，API Monitor是一个免费软件，可以让你监视和控制应用程序和服务，取得了API调用
![在这里插入图片描述](https://img-blog.csdnimg.cn/a1b7df22e09840c19fade3893ad05a14.png)
导入我们的恶意软件
![在这里插入图片描述](https://img-blog.csdnimg.cn/ce8b44f84c494a1b96397d5ba8d5a955.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/e110e6179a574d5bbec7068dd7b4881e.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/7b01a296cacd496fb4fcfe572dd7189a.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/d5f6a53a1bc44d2c9d1daddf5e1c543f.png)
这里记录了恶意软件正在使用的所有windowsAPI，我们可以在这里进行跟踪
我打算挖掘一下这个恶意软件创建的文件
ctrl+f
![在这里插入图片描述](https://img-blog.csdnimg.cn/a1516aecdd5543cd9400d89b5cb335a5.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/c989a4b93c594db387540399135d1cfb.png)
我们进入这个目录下，对browserassist.dll文件进行分析
![在这里插入图片描述](https://img-blog.csdnimg.cn/d749d69893e643b6b1fe09a4544c3bf2.png)
将此dll文件拖入peid进行分析
![在这里插入图片描述](https://img-blog.csdnimg.cn/be15b1a6ecda41cdb2b92ec08fdae0e4.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/31698279ea7841dbb1ea1d0d46a7142e.png)
看起来只是一个普通的二进制文件，额外的信息显示，此文件也没有被加密混淆
我们使用ida进行分析，交互式反汇编器专业版（Interactive Disassembler Professional），人们常称其为IDA Pro，或简称为IDA。是最棒的一个静态反编译软件，为众多0day世界的成员和ShellCode安全分析人士不可缺少的利器，IDA Pro是一款交互式的，可编程的，可扩展的，多处理器的，交叉Windows或Linux WinCE MacOS平台主机来分析程序， 被公认为最好的花钱可以买到的逆向工程利器
![在这里插入图片描述](https://img-blog.csdnimg.cn/dca37dc2b3024390a008f65dfbf8726e.png)
ctrl+f12看一下程序里的字符串，我们可以看到，这里对http标头值进行了引用，Content-Type，Accept-Encoding，POST,GET等，可以进一步的确定，他对浏览器执行了某些操作
![在这里插入图片描述](https://img-blog.csdnimg.cn/95bb8b6a6ddc4af8b130713ec9175caa.png)
现在比较疑惑的是，为什么将这个dll文件放在了IE的目录下，可能感染了IE，我们打开IE看看
ie的右上角有一个笑脸
![在这里插入图片描述](https://img-blog.csdnimg.cn/1c05e41740014107a53d30d1e95e7681.png)
我们点击发送笑脸看看
![在这里插入图片描述](https://img-blog.csdnimg.cn/448b88e53d4c4a3a85bcdd48e50a07aa.png)
他似乎伪造了一个ie的反馈表，我们看看隐私声明
![在这里插入图片描述](https://img-blog.csdnimg.cn/6ef455dc739a4623a451790585476cbf.png)
看起来是一个正常的隐私声明
我们去google看看
```
intermet explorer smiley
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/eda1d5a2fc0f4ba496405b534e3a57b4.png)
emm....这不是一个恶意软件吗，难道这个笑脸实际就是在ie中....
重新分析一下，从最上面的分析详细信息看，这个恶意软件是一个浏览器助手的安装程序，也许我们应该看看火狐浏览器，考虑到软件是2018年的，我去下载了一个老版本的火狐
```
https://ftp.mozilla.org/pub/firefox/releases/40.0/win32/zh-CN/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2ed6efe4fed54028afe104a32c69b53c.png)
安装完成后，打开老版本的火狐
![在这里插入图片描述](https://img-blog.csdnimg.cn/c9fd6ee9d173492dad5a5fb0c56b1c0b.png)
这个恶意软件没有明显的特征，也许是在做一些偷偷摸摸的事，接下来，我尝试跟踪对browserassist.dll的调用，以防火狐对其进行任何处理
回到API Monitor，我们将外部dll添加到API监视器
![在这里插入图片描述](https://img-blog.csdnimg.cn/52450db603d64b52936550963726924b.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/8f8fa749f2124cbe822d27b3987bfa0a.png)
但是它说它不会导出任何函数，所以这个.dll文件不像具有导出函数的典型.dll文件那样运行
![在这里插入图片描述](https://img-blog.csdnimg.cn/7a11e6098b2e4fb4b99ed7fff0706478.png)
接下来我使用x32dbg进行分析，这个工具是专业的动态调试器，我打ctf和做逆向分析时经常用
位置在：
```
C:\Program Files\x64dbg\release\x32
```
打开软件
alt+a选择附加程序
![在这里插入图片描述](https://img-blog.csdnimg.cn/dccd1543ff2f402f8a627050e9e2db94.png)
我们选择火狐
![在这里插入图片描述](https://img-blog.csdnimg.cn/227a21713baa47c19019cad3559e7a06.png)
我们查看一下内存布局
![在这里插入图片描述](https://img-blog.csdnimg.cn/27244d57ed51457d9b3a6cb04a4d2273.png)
我们在这里找到了那个dll文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/02fbe28aedd5419caad56cb31904f198.png)
显然我们感兴趣的是text段，这里是可执行代码存放的地方
![在这里插入图片描述](https://img-blog.csdnimg.cn/3a6606ffbc3b4690893b9542c57432f0.png)
双击browserassist.dll的text段，我们跳转到代码执行的位置
![在这里插入图片描述](https://img-blog.csdnimg.cn/7ea585216b7f4c41a0329652e7f58836.png)
然后按下f2，在dll入口处的开头和结尾下一个断点来进一步分析
![在这里插入图片描述](https://img-blog.csdnimg.cn/ffa292e3e16f464b92c9ca5df01c2950.png)
重新载入一下程序
![在这里插入图片描述](https://img-blog.csdnimg.cn/a2fb252e241b495f8bfa3f2187462ed1.png)
我们一直按f9慢慢分析程序，有一段代码被加载了许多次，而且我注意到一些ascii字符串在堆栈中被引用过，右下角的窗口是程序的堆栈区
最终我注意到了这个json数据，它在浏览器里注入了一些东西
![在这里插入图片描述](https://img-blog.csdnimg.cn/ed49691a38de4095a2a21b6a678bc323.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/658e9e9c0c384617947429411ba32008.png)
理论上来讲，如果这是平时真正的恶意软件的话，这个可能是注入广告或者脚本来窃取隐私信息
# 总结
这个是国外前几年的一个ctf比赛的题，看起来还不错，学习到一些恶意软件分析取证的方法，还是收获颇多的，现在在这里记录一下做题的笔记





















