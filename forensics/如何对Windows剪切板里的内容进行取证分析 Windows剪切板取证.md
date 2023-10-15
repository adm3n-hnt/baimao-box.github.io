![在这里插入图片描述](https://img-blog.csdnimg.cn/6cba9013fb1c449fbe6ce75ff3abd4e0.jpeg)

# 前言
无论是在现实中对设备进行取证分析，还是在ctf中做取证类的题目，剪切板里的内容都需要去查看，以免遗漏什么重要信息

# 剪切板位置
剪切板是计算机操作系统提供的一个临时存储区域，用于在不同应用程序之间复制和粘贴文本、图像和其他数据。剪切板通常位于操作系统的内存中，而非直接可见

但是windows剪切板的内容都被加密存储在这个文件夹下
```
C:\Users\[your user name]\AppData\Local\Microsoft\Windows\Clipboard\Pinned\
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/4fe7fbc35c72482b8f0304c03841deec.png)

在这个文件夹下有一个GUID名的文件夹，GUID 是一个由数字和字母组成的标识符，用于唯一标识对象、组件或资源。在 Windows 中，GUID 通常用于标识注册表项、文件夹、设备驱动程序等。每个 GUID 都是唯一的，几乎不可能发生重复

进入这个文件夹，可以看到还有一个GUID名的文件夹和一个json文件，打开这个json文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/31506a4be55d4f75b95f36761d701b33.png)
```
{
  "items": {
    "{773AC540-7E10-41F1-B0EE-22D82BCDD303}": {
      "timestamp": "2023-03-27T22:35:58Z",
      "source": "Local",
      "cloud_id": "8F58ABBE-4C70-EAAB-AD53-3EA484D6C95D"
    }
  }
}
```
上述 JSON 数据表示一个包含 "items" 键的对象，该键对应的值是另一个对象。内部对象具有一个键 "{773AC540-7E10-41F1-B0EE-22D82BCDD303}"，该键对应的值是一个包含 "timestamp"、"source" 和 "cloud_id" 键的子对象。

子对象中的 "timestamp" 键对应的值是 "2023-03-27T22:35:58Z"，表示时间戳。"source" 键对应的值是 "Local"，表示来源信息。"cloud_id" 键对应的值是 "8F58ABBE-4C70-EAAB-AD53-3EA484D6C95D"，表示云标识符

进入文件夹，可以看到三个文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/1e605f7b32f249c49c02b1476a6bc948.png)

一个是json文件，另外两个文件名都是经过base64加密过的

![在这里插入图片描述](https://img-blog.csdnimg.cn/b56cb08ccaa94a5194ace907eeb1fb90.png)

但是这些文件内的内容都被加密混淆了，不知道key就无法复原

![在这里插入图片描述](https://img-blog.csdnimg.cn/29d7bf2cd057485a8be00039c9ee3569.png)

# 剪切板取证
自 Windows 10 版本 1803 以来，ActivitiesCache.db 已开始记录剪贴板活动，ActivitiesCache.db的位置在
```
%AppData%\Local\ConnectedDevicesPlatform\[user]\
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/efa73ddeee2a40d68f69697fdfcdb3ea.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/669c52b7168940a1a98f4a8f9ef7045e.png)

在这个数据库里，存储着剪切板里的内容，我们需要安装sqlitebrowser工具才能查看数据库

```
apt install sqlitebrowser
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/62acaf94079a46048cc2c08d3d6e2b5a.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/0e6545d15af949df92280930086c32f5.png)

点击browse data，选择activityoperation表

![在这里插入图片描述](https://img-blog.csdnimg.cn/0afdec8676f54beea81e3eea57e426fe.png)

点击cipboardpayload列进行排序

![在这里插入图片描述](https://img-blog.csdnimg.cn/e43ad1963f654ed18bf6237aa59959aa.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/4edbbd6f9f2f4a06b65afd4382e0bd80.png)

这些数据就是剪切板里的内容，数据经过了base64加密，我们解密即可

![在这里插入图片描述](https://img-blog.csdnimg.cn/d99347803c004abab5c7fe00c61573f1.png)

成功获取到剪切板里的内容

# 其他
生成ActivitiesCache.db文件需要启用剪贴板历史记录

![在这里插入图片描述](https://img-blog.csdnimg.cn/9274a11c7db5406cbd7a689a825a900e.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/6c34451d8ccc494cbf94428f14c9f2c2.png)

