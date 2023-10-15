# ARM 编程模拟器
ARM 编程模拟器网站地址：
```
https://cpulator.01xz.net/?sys=arm-de1soc
```
# 硬件交互
在这个模拟器的右边有各种不同的硬件设备，这里演示两个硬件设备

![在这里插入图片描述](https://img-blog.csdnimg.cn/d81c2ef28fcc40e7bad68d65c7ccd5be.png)

switches是输入硬件，LED是输出硬件，我们可以写一个小程序让他执行

首先我们需要声明一个变量来存储switches的位置以及LED的位置，上面标注了硬件的位置

![在这里插入图片描述](https://img-blog.csdnimg.cn/b340ac4a14a3409995a04c25ccf0d14f.png)

首先我们先调用switches
```
.equ switch, 0xff200040 
.global _start
_start:
	ldr r0,=switch
```
但我们这样直接存储位置的话，只是一个普通的变量，并没有什么用

![](https://img-blog.csdnimg.cn/0be459b65a5c4c5e92d03fe62eff0375.png)

如果我们想用这个寄存器，我们可以打开switches上的开关，这里的意思是2的一次方，二次方……

![在这里插入图片描述](https://img-blog.csdnimg.cn/df0934b477c3430b8059457ecbbb8058.png)

如果我们选择了1这个开关，计算的时候就会有变化
```
.equ switch, 0xfff200040 
.global _start
_start:
	ldr r0,=switch
	ldr r1,[r0]
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/9e9731830a3b43ba8b7f0ab860a844b5.png)
可以看到，r1寄存器里的值是2，因为2的一次方是2，我们选择其他选项试试

![在这里插入图片描述](https://img-blog.csdnimg.cn/1deb28ccece4478e9ba3ce251bad87c3.png)

可以看到r1寄存器里的值为7，因为2的零次方+2的一次方+2的二次方

这是从硬件获取输入的一个简单的演示
硬件第一排的LED最右边的代表二进制的第一个数值，然后是第二个数值，……以此类推
```
.equ switch,0xff200040 
.equ led,0xff200000
.global _start
_start:
	ldr r0,=switch
	ldr r1,[r0]
	
	ldr r0,=led   //覆盖r0之前的地址，变成led的地址
	str r1,[r0]  //str：字数据存储指令，因为LED是输出硬件，我们需要把值存入后才能输出
```

执行程序，可以看到LED的灯亮起，红的是1，不亮的则为0，因为r1寄存器里的值十进制为7，二进制就为0111

![在这里插入图片描述](https://img-blog.csdnimg.cn/f01f03e7edc84280a79191302a0344d6.png)

# 为 ARM 设置 Qemu

现在我们要在linux搭建运行arm的环境
首先下载这个文件，这可以模拟操作系统环境，还有其他版本的环境，目前这个版本和qemu工具运行是最稳定的：
```
https://downloads.raspberrypi.org/raspbian/images/raspbian-2017-04-10/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/194e8564bc3544e2bc20c81521b42c22.png)

然后需要一个内核：
```
https://github.com/dhruvvyas90/qemu-rpi-kernel/blob/master/kernel-qemu-4.4.34-jessie
```
下载好后放到同一个文件夹下

![在这里插入图片描述](https://img-blog.csdnimg.cn/e0087f36ba8c44a095f0eba8c3621767.png)

然后安装qemu
```
apt-get install qemu-system
```

安装完成后执行这一条命令来创建环境
```
qemu-system-arm -kernel kernel-qemu-4.4.34-jessie -cpu arm1176 -m 256 -M versatilepb -serial stdio -append "root=/dev/sda2 rootfstype=ext4 rw" -hda 2017-04-10-raspbian-jessie.img -nic user,hostfwd=tcp::5022-:22 -no-reboot
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/8824e4951ae442648fe46e81afd014ba.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/e7ae77997af14433962c5a30123a025f.png)


现在树莓派虚拟环境就安装好了，打开树莓派里的终端，输入命令打开ssh
```
sudo service ssh start
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/7865a3c7f66b431e822253735c11e1cb.png)

然后在本地终端里连接ssh
```
ssh pi@127.0.0.1 -p 5022
```
输入yes，密码为raspberry

![在这里插入图片描述](https://img-blog.csdnimg.cn/d44028b7c96749e882ac1d9db180007c.png)
# 总结
这是我学习的笔记，有什么错误和不懂的地方欢迎来私信我，或者加我qq

