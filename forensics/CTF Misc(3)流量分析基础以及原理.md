![在这里插入图片描述](https://img-blog.csdnimg.cn/5eb8a1785917493bb5f0d5573434e1f0.png)
# 前言
流量分析在ctf比赛中也是常见的题目，参赛者通常会收到一个网络数据包的数据集，这些数据包记录了网络通信的内容和细节。参赛者的任务是通过分析这些数据包，识别出有用的信息，例如登录凭据、加密算法、漏洞利用等等

# 工具安装
Wireshark是一款开源的网络数据包分析工具，用于捕获、分析和可视化网络流量。它在多个平台上可用，包括Windows、Mac和Linux，工具下载地址
```
https://www.wireshark.org/download.html
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/428b96f4a17a4a75afe0ddc34d23e965.png)

默认安装即可
# 工具介绍
## 查看捕获到的数据包列表

演示的流量包：
```
https://anonfiles.com/laS2l3v2zd/Alpha_1_pcapng
```
访问网址下载即可

![在这里插入图片描述](https://img-blog.csdnimg.cn/0c12e3046f4a4d6ca75b2c1f01ab4f37.png)

这是流量包的文件格式
```
.pcapng
```
安装好wireshark后，双击打开这个文件

最上方的一栏是任务栏，下面是搜索框，最下面的是数据包的详细信息和数据包的数据

![在这里插入图片描述](https://img-blog.csdnimg.cn/1df4737a07a84a61ab6bca071f2946c3.png)

##  常用过滤器命令和语法
协议筛选：
```
tcp：显示所有TCP协议的数据包
udp：显示所有UDP协议的数据包
http：显示所有HTTP协议的数据包
dns：显示所有DNS协议的数据包
icmp：显示所有ICMP协议的数据包
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/4f189ec17d304b7cb3becd793d0af983.png)

在搜索框里输入http就能看到所有的http流量，其他同理

IP地址筛选：
```
ip.addr == 192.168.0.1：显示与指定IP地址相关的所有数据包
src host 192.168.0.1：显示源IP地址为指定地址的数据包
dst host 192.168.0.1：显示目标IP地址为指定地址的数据包
```
这里我们筛选192.168.1.25ip相关的数据包

![在这里插入图片描述](https://img-blog.csdnimg.cn/2d5d1ab88a204d8e9b50cbe4372984e5.png)

筛选过后就是都关于这个ip的数据包流量

![在这里插入图片描述](https://img-blog.csdnimg.cn/5f449211eaa94c8782a5da7df442b067.png)



端口筛选：
```
tcp.port ==  80：显示使用指定TCP端口的数据包
udp.port == 53：显示使用指定UDP端口的数据包
port 80：显示源或目标端口为指定端口的数据包
```
筛选tcp 80端口的流量

![在这里插入图片描述](https://img-blog.csdnimg.cn/ce92fd12d9d24144849beedaae814d25.png)


逻辑运算符：
```
and：使用AND逻辑运算符连接多个条件，例如 tcp and ip.addr == 192.168.0.1
or：使用OR逻辑运算符连接多个条件，例如 tcp or udp
not：使用NOT逻辑运算符排除满足条件的数据包，例如 not tcp
```
 筛选tcp流量和ip地址为192.168.1.5的流量包tcp and ip.addr == 192.168.1.5

![在这里插入图片描述](https://img-blog.csdnimg.cn/a797c557adfe40379a07899223674d6d.png)

比较运算符：
```
==：等于，例如 http.request.method == "POST"
!=：不等于，例如 ip.addr != 192.168.0.1
<、>：小于、大于，例如 tcp.len > 100
```
筛选http的post请求流量包http.request.method == "POST"

![在这里插入图片描述](https://img-blog.csdnimg.cn/b995f9d3150c4df4abda85136c175c57.png)


复杂筛选：
```
使用括号 () 来组合多个条件，例如 (tcp and port 80) or (udp and port 53)
使用复合条件进行筛选，例如 (tcp.flags.syn == 1 or tcp.flags.ack == 1) and ip.addr == 192.168.0.1
```


# HTTP流量分析
演示的流量包：
```
https://anonfiles.com/laS2l3v2zd/Alpha_1_pcapng
```
访问网址下载即可

![在这里插入图片描述](https://img-blog.csdnimg.cn/0c12e3046f4a4d6ca75b2c1f01ab4f37.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/af3176b6d6944c02a08de1256c4ee811.png)

点击协议分级，可以看到这个流量包里所有协议的流量

![在这里插入图片描述](https://img-blog.csdnimg.cn/06da3bddc1a042c0b4bda2e76183088f.png)

有udp流量，ipv4流量，tcp流量，http流量，我们选择http流量数据，右击选择选中

![在这里插入图片描述](https://img-blog.csdnimg.cn/de48c662b22b46cc9671d4c755c8af0f.png)



![在这里插入图片描述](https://img-blog.csdnimg.cn/92affb55a5b6471bb6561783eecb5104.png)


就可以看到这个流量包里的所有http流量

我们随便选择一个包，跟踪他的http流量，以便查看详细数据

![在这里插入图片描述](https://img-blog.csdnimg.cn/b26ad91737854cee9c97d883f6fecad8.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/488478bfc61e4ce0acda53055f0745fc.png)

在下面，很明显能看到sql注入的流量

![在这里插入图片描述](https://img-blog.csdnimg.cn/3ad9c448a40e46b3bef23baa58434aa7.png)


之后他上传了自己的php一句话木马并连接执行命令

![在这里插入图片描述](https://img-blog.csdnimg.cn/03d4f903a0944fceaac3ee723def373d.png)

右击跟踪这个流量包

![在这里插入图片描述](https://img-blog.csdnimg.cn/510cba8111474dafa8402b8d1a4cdf20.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/c36ecc96a55f4d65a31d14b55c61aa5d.png)

这个一句话木马将执行的内容加密了，使用的是base64编码，我们复制加密后的参数，进入网站解密

```
https://base64.us/
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/ea7ccc0b7d784539b8287fddce32b632.png)

他查询了C:\\phpStudy\\WWW\\目录下的文件，我们可以看到回显

![在这里插入图片描述](https://img-blog.csdnimg.cn/022637118b2445dcbefd47a9d22791ea.png)


点击右下角的返回，继续跟踪流量包

![在这里插入图片描述](https://img-blog.csdnimg.cn/4a900a8a9873447c8c128e12ec3cfc85.png)

在最后一个用一句话木马的流量里，可以看到它导出了一个叫flag.zip的文件


![在这里插入图片描述](https://img-blog.csdnimg.cn/81eeff1e3d12465cb555b41b89d99dce.png)

由于文件未加密，我们还能看到flag.txt的内容

![在这里插入图片描述](https://img-blog.csdnimg.cn/4f0c15aa492e4ad79f6d38d4cdd24fb7.png)

flag.txt文件内的文本为：

```
DPS
```

# DNS流量分析

题目下载地址：
```
https://anonfiles.com/S5Fc91v3za/capture_pcap
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/9c0de464a5b04ef8a59bddd7fade708e.png)

打开流量包，可以看到全是DNS流量，使用strings工具可以发现很多十六进制

![在这里插入图片描述](https://img-blog.csdnimg.cn/16797a819d2b4a61a2cf577584e10b99.png)


我们把这些十六进制导出来

```
tshark -r capture.pcap -T fields -e dns.qry.name > a.txt
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/ebd8cc29b19141c99bf28fbc2023c2aa.png)

用文本编辑器把.pumpkincorp.com字符去掉

![在这里插入图片描述](https://img-blog.csdnimg.cn/53fc96ab550945d18c144848761c34e7.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/700070e2af1f45379c8cd0ebbd467c60.png)

然后再用uniq工具将重复的字符串去掉
```
cat a.txt| uniq > b.txt
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/fac043b002494b8190791b89dd905975.png)
将十六进制转换为ascii码可以发现，这是一个Excel 文件

```
https://gchq.github.io/CyberChef/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/3fc270c8be6a445d96e900cc00022efd.png)

可以看到flag文本


![在这里插入图片描述](https://img-blog.csdnimg.cn/e32e94dd144e42d290c6f888b09bd41f.png)

# 键盘流量分析

最常见的usb键盘流量包如下图

![在这里插入图片描述](https://img-blog.csdnimg.cn/c2f560e421044e5b9a1703efae8a775a.png)

协议为USB，并且键盘数据存储在usbhid.data中，这里0c对应的就是i字符

![在这里插入图片描述](https://img-blog.csdnimg.cn/b99c3302221a41818209e6c542850443.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/780da3d0e1de4a72819bf1b59fdca88f.png)



我们将流量包里的usbhid.data数据提取出来，然后一一和字符对应即可，这里我开发了一个脚本，可以直接提取和转换usb键盘流量数据

~~~
https://github.com/baimao-box/KeyboardTraffic
~~~
![在这里插入图片描述](https://img-blog.csdnimg.cn/c36df1a3019948e9aeb72f390c79f038.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/5e437e65e54747b6a5b90ec42c88f867.png)

# 总结
这篇文章我只是展示了一些流量分析的基础，想要成为大佬，就要多刷题

