
# 开始
![在这里插入图片描述](https://img-blog.csdnimg.cn/e5a335a442824a9da580498fa0c29a7f.png)

通过file工具可以知道，这是一个64为可执行程序，并且开启了stripped

将文件导入Ghidra进行逆向分析

![在这里插入图片描述](https://img-blog.csdnimg.cn/4153627bdca4404b9c3debb5cc612e8d.png)

定位到main函数地址

![在这里插入图片描述](https://img-blog.csdnimg.cn/e6915f2a9328442ab96377aa82050a7f.png)
```
undefined8 FUN_00101427(void)

{
  int iVar1;
  long in_FS_OFFSET;
  char local_48 [16];
  undefined local_38 [40];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  printf("Welcome to baby\'s first rev! :>\nPlease enter your username: ");   #输出字符“Please enter your username:”
  __isoc99_scanf(&DAT_00102045,local_48);   #获取输入
  printf("Please enter your password: ");   #输出字符“Please enter your password:”
  __isoc99_scanf(&DAT_00102045,local_38);    #获取输入
  iVar1 = strcmp(local_48,"bossbaby");   #将local_48地址里的值与bossbaby字符串做对比
  if (iVar1 != 0) {    #输入不正确
    printf("%s? I don\'t know you... stranger danger...",local_48);
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  puts("You\'re almost there!");
  iVar1 = FUN_001012b2(local_38);  #对local_38里的值执行了FUN_001012b2操作
  if (iVar1 == 0x26) {   #如果iVar1 == 0x26
    printf("You\'re boss baby!");  #输出You\'re boss baby!
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {  #输入不正确
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```
即使输入正确也没有输出flag

从上面分析可以知道，用户名是bossbaby，而密码是flag

我们去到FUN_001012b2地址

![在这里插入图片描述](https://img-blog.csdnimg.cn/db34611ecf1f486fa6aca09d610703b0.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/a0c3d9901ce64838ad00299fd4e05375.png)


在代码的底部，我们看到了一个DAT_00104020变量，它指向存储在二进制文件中的数据，我们可以双击它来查看它包含的内容

![在这里插入图片描述](https://img-blog.csdnimg.cn/c8158813255e4d3e848a50ca79cbb75c.png)

这里有很多的十六进制值，将他们导出到python

```
data = b'\x66\x00\x00\x00\xd9\x00\x00\x00\x88\x01\x00\x00\x41\x03\x00\x00\xc0\x07\x00\x00\xf9\x06\x00\x00\xa4\x18\x00\x00\x95\x00\x00\x00\x0a\x01\x00\x00\xd5\x01\x00\x00\x7c\x03\x00\x00\xa9\x03\x00\x00\xb0\x07\x00\x00\x69\x19\x00\x00\x27\x01\x00\x00\xa3\x01\x00\x00\xc4\x01\x00\x00\xb9\x02\x00\x00\x54\x07\x00\x00\x89\x08\x00\x00\x50\x0f\x00\x00\xf0\x01\x00\x00\x54\x02\x00\x00\xd9\x02\x00\x00\x58\x05\x00\x00\x71\x05\x00\x00\x24\x09\x00\x00\x19\x10\x00\x00\x42\x03\x00\x00\xad\x03\x00\x00\x08\x05\x00\x00\xe9\x06\x00\x00\x30\x0a\x00\x00\xe1\x10\x00\x00\x84\x12\x00\x00\x00\x05\x00\x00\xd2\x05\x00\x00\x4d\x07\x00\x00'
```

看起来像是以 4 个字节分隔的。在\x00 4个字节之后，我们不断看到一个值、一些字符，然后是另一个值。我们去伪代码里看一下这些数据会发生什么

![在这里插入图片描述](https://img-blog.csdnimg.cn/6941a1f0b7d6424995751c44b3736f61.png)

在while(true)的循环中，我们看到我们有一个local_48变量，每次都会增加1
```
with local_48 = local_48 + 1
```
所以我们可以重命名i

![在这里插入图片描述](https://img-blog.csdnimg.cn/572069a3b9004eafb892b18fcfada498.png)

然后我们看到在if()语句中它也使用了这个i变量。i * 4被添加到 和 的地址中DAT_00104020进行local_38比较。这符合我们上面的猜想，这些值每个是 4 个字节

因此，如果将local_38变量与数据进行比较，则数据可能是加密标志，并且local_38变量是我们提供的加密密码，我们重命名local_38变量为encrypted_password


![在这里插入图片描述](https://img-blog.csdnimg.cn/d3b7b714a8ad4289a81c93312e5a5d3a.png)
在上面encrypted_password被设置为一个地址
```
encrypted_password = (undefined *)((long)psVar5 + lVar1);
```
(long)psVar5 + lVar1也用于下面函数中
```
FUN_00101209(pcVar2,(long)psVar5 + lVar1);
```
因此FUN_00101209，第一个参数是我们给定的密码，第二个参数是encrypted_password与数据进行比较的地址

我们双击FUN_00101209

![在这里插入图片描述](https://img-blog.csdnimg.cn/d862bffa4d59416db0894a53cc14fe00.png)

这是另一个while(true)循环

我重命名了一下，看起来更简洁点

![在这里插入图片描述](https://img-blog.csdnimg.cn/beaa4629be844f21a36702524f860966.png)

我们可以写一个python脚本来还原flag

```
import struct

data = b'\x66\x00\x00\x00\xd9\x00\x00\x00\x88\x01\x00\x00\x41\x03\x00\x00\xc0\x07\x00\x00\xf9\x06\x00\x00\xa4\x18\x00\x00\x95\x00\x00\x00\x0a\x01\x00\x00\xd5\x01\x00\x00\x7c\x03\x00\x00\xa9\x03\x00\x00\xb0\x07\x00\x00\x69\x19\x00\x00\x27\x01\x00\x00\xa3\x01\x00\x00\xc4\x01\x00\x00\xb9\x02\x00\x00\x54\x07\x00\x00\x89\x08\x00\x00\x50\x0f\x00\x00\xf0\x01\x00\x00\x54\x02\x00\x00\xd9\x02\x00\x00\x58\x05\x00\x00\x71\x05\x00\x00\x24\x09\x00\x00\x19\x10\x00\x00\x42\x03\x00\x00\xad\x03\x00\x00\x08\x05\x00\x00\xe9\x06\x00\x00\x30\x0a\x00\x00\xe1\x10\x00\x00\x84\x12\x00\x00\x00\x05\x00\x00\xd2\x05\x00\x00\x4d\x07\x00\x00'

encrypted_values = [unpacked[0] for unpacked in struct.iter_unpack('<I', data)]
flag = b""
for i in range(len(encrypted_values)):  
    for c in range(256):  
        value = i * i + (c << (i + (i // 7) * -7 & 0x1f))
        if value == encrypted_values[i]: 
            flag += bytes([c])
            break

print(flag)
```
执行文件，获得flag

![在这里插入图片描述](https://img-blog.csdnimg.cn/2c96a5d3e5194812b7aeaf08cecf7133.png)

