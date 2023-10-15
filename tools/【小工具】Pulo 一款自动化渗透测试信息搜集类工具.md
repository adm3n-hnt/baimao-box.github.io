# Pulo v1.0
项目地址：
```
https://github.com/baimao-box/Pulo
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/ae7efa084b1c49bf860fef3f764d1c30.png)

这是一个用于渗透测试的自动化信息收集工具。只需输入域名或ip，即可获取模板的开放端口信息、web目录、子域名、插件漏洞等信息。

该工具还可以根据扫描到的端口服务版本和输出自动在漏洞库中搜索相关漏洞

# 安装
git：
```
git clone https://github.com/baimao-box/Pulo.git
```
进入解压后的目录
```
chmod +x setup.py
python3 setup.py
```
需要等待几分钟，期间需要手动选择一些东西，默认即可

![在这里插入图片描述](https://img-blog.csdnimg.cn/690235ad4a0c4cae88819464e52dbbae.png)

安装完成后会提示是否安装searchsploit。这个工具很大，所以在这里问。如果网络好，可以安装。

# 运行
```
help:
pilo -u [url]
example:
pilo -u http://www.exploit.com
```
工具运行需要一些时间，您需要等待5分钟左右

![在这里插入图片描述](https://img-blog.csdnimg.cn/89e0db76f45e44c9a3503e49bb6d5624.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/a47f28602ad24a41af51f2f69fd8ab05.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/889ec155d4e34b8db9b7ffff454874cf.png)


只需要输入域名，即可获取开放端口和服务版本、web目录、子域名、插件漏洞信息

还可以根据扫描到的服务版本自动在Kali的exploit-db漏洞库中搜索相关漏洞
