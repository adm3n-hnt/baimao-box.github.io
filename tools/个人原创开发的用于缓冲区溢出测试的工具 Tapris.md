# Tapris v1.0
![在这里插入图片描述](https://img-blog.csdnimg.cn/84c36211b1604e88b3a09c7d3ba87c89.png)
一个用于缓冲区溢出测试的工具

github项目地址：https://github.com/baimao-box/tapris

![在这里插入图片描述](https://img-blog.csdnimg.cn/a3cff5fd519b431b9b7059a688d6caf7.png)

# 运行
```
./tapris.py
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/882c43bc86dd4bb891b9e8cbb6c49189.png)
# 模块解释
```
更改目标ip地址和端口
目标程序缓冲区溢出测试
获取程序溢出边界数值
排除坏字符
pwn掉程序，获取回连shell
```
# 演示
![在这里插入图片描述](https://img-blog.csdnimg.cn/ef612c97a4c74b6e97b0a6e443cca0ab.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/095706265ada4ca98b6a07bc2b4469ac.png)

在我的github上有演示：https://github.com/baimao-box/tapris


# 其他
工具在运行时可以正常执行linux命令

![在这里插入图片描述](https://img-blog.csdnimg.cn/fb27f26f90ee40eda1ebc6c8de52d831.png)

在测试程序坏字符时，如果程序坏字符很多，可以重复测试，很方便
