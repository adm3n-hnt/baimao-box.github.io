# 简介
需要题目和逆向工具的都可以加我qq：3316735898，有什么不懂的也可以来问我
# 开始
将题目拖入工具后，定位到main函数，这里因为ida看伪代码有点头大，这里我就用ghidra来逆向题目

![在这里插入图片描述](https://img-blog.csdnimg.cn/e807b7a4f81f4575b21872b861650c3b.png)
伪代码：

![在这里插入图片描述](https://img-blog.csdnimg.cn/92c1308333b742ad97214a546c06b483.png)
非常简单就能看出这个程序做了什么
```
void main(void)

{
  int iVar1;
  undefined8 extraout_RDX;
  long lVar2;
  size_t *__n;
  EVP_PKEY_CTX *ctx;
  long in_FS_OFFSET;
  char *local_20;
  size_t local_18;
  undefined8 local_10;
  
  local_10 = *(undefined8 *)(in_FS_OFFSET + 0x28);
  printf("Enter killswitch: ");          //输出字符串Enter killswitch
  local_18 = 0;
  __n = &local_18;
  local_18 = getline(&local_20,__n,stdin);    //读取我们输入的值，然后存放到local_18变量里
  local_20[local_18 - 1] = '\0';     //对我们输入的值做了一些操作
  puts("Verifying authentication...");    //输出字符串Verifying authentication...
  iVar1 = checker(local_20);       //检查我们输入的字符，然后将布尔值放到iVar1函数里
  if (iVar1 == 1) {            //如果iVar1 == true 则：
    puts("Correct authentication received, shutting down");       //输出字符串Correct authentication received, shutting down
    func_0x00400580(0);    //我跟进函数看了一下，是退出程序的操作，类似与exit()
  }
  puts("ERROR - Invalid authentication!");     //如果我们输入的值不正确，则输出ERROR - Invalid authentication，很明显，我们这是我们输入错误得到的字符串
  ctx = (EVP_PKEY_CTX *)0xffffffff;
  func_0x00400580();     //我跟进函数看了一下，是退出程序的操作，类似与exit()
  _init(ctx);
  lVar2 = 0;
  do {
    (*(code *)(&__frame_dummy_init_array_entry)[lVar2])((ulong)ctx & 0xffffffff,__n,extraout_RDX);
    lVar2 = lVar2 + 1;
  } while (lVar2 != 1);
  return;
}
```
通过分析程序可以发现，我们无论输入正确或者错误都不会显示flag，而flag应该在程序里，需要通过一些操作才能还原flag

![在这里插入图片描述](https://img-blog.csdnimg.cn/164808bc87664f7e9f8c70476ae38033.png)


checker函数是关键所在，他存放着flag的值，与我们输入的字符串做对比，输入正确则显示成功的字符串，输入错误则显示错误的字符串
 
 回到ghidra，双击进入checker函数

![在这里插入图片描述](https://img-blog.csdnimg.cn/a3d8e83dc21042398e7af1ea503780bd.png)

可以看出，这个函数做了while循环
```
  local_3c = 0;
  do {        //进入while循环
    if (29 < local_3c) {      //if判断，如果29 < 0
      uVar1 = 1;    //uVar1的值为1
code_r0x0040081c:  	//循环
      if (*(long *)(in_FS_OFFSET + 0x28) == *(long *)(in_FS_OFFSET + 0x28)) {   //对比偏移量里的值，相同则：
        return uVar1;   //返回uVar1的值，也就是true
      }   //但这个条件不会发生，因为29 < 0是不正确的
                    /* WARNING: Subroutine does not return */
      __stack_chk_fail();
    }
    if (*(char *)(param_1 + (int)local_3c) != check[(int)(local_3c * 5)]) {  //param_1里的值+local_3c里的值不等于check里的值*5则：
      uVar1 = 0xffffffff;  //uVar1的值为0xffffffff
      goto code_r0x0040081c;   //然后跳转此处，也就是第5行，一直循环
    }   //这个才是我们循环正常进行的地方
    local_3c = local_3c + 1; //叠加
  } while( true );
}
```
我们需要uVar1 = 1，这样我们才能满足main函数的条件，执行正确的输出，我们双击这个关键的check

![在这里插入图片描述](https://img-blog.csdnimg.cn/afdcd1dcb65d4b9181b591fabd88775c.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b76d28623bdd4772a7b89d09eaedc8a3.png)
里面存储了一大堆的十六进制数，我们还原这个check里的数就能得到flag了
我们将里面的十六进制值复制出来，然后写一个脚本还原flag
```
data = [
    0x48,
    0x42,
    0xEC,
    0x99,
    0xEA,
    0x54,
    0xEC,
    0x99,
    0xEA,
    0xC2,
    0x42,
    0x99,
    0xEA,
    0xC2,
    0x52,
    0x7B,
    0xEA,
    0xC2,
    0x52,
    0x2A,
    0x34,
    0xC2,
    0x52,
    0x2A,
    0x09,
    0x5F,
    0x52,
    0x2A,
    0x09,
    0x7D,
    0x72,
    0x2A,
    0x09,
    0x7D,
    0x81,
    0x34,
    0x09,
    0x7D,
    0x81,
    0xC6,
    0x74,
    0x7D,
    0x81,
    0xC6,
    0x2A,
    0x68,
    0x81,
    0xC6,
    0x2A,
    0xE9,
    0x33,
    0xC6,
    0x2A,
    0xE9,
    0xF6,
    0x72,
    0x2A,
    0xE9,
    0xF6,
    0x21,
    0x5F,
    0xE9,
    0xF6,
    0x21,
    0xAD,
    0x30,
    0xF6,
    0x21,
    0xAD,
    0x62,
    0x66,
    0x21,
    0xAD,
    0x62,
    0x00,
    0x66,
    0xAD,
    0x62,
    0x00,
    0xBD,
    0x62,
    0x62,
    0x00,
    0xBD,
    0xCF,
    0x33,
    0x00,
    0xBD,
    0xCF,
    0x59,
    0x34,
    0xBD,
    0xCF,
    0x59,
    0x1C,
    0x74,
    0xCF,
    0x59,
    0x1C,
    0x7C,
    0x5F,
    0x59,
    0x1C,
    0x7C,
    0x90,
    0x62,
    0x1C,
    0x7C,
    0x90,
    0x50,
    0x31,
    0x7C,
    0x90,
    0x50,
    0x5D,
    0x6E,
    0x90,
    0x50,
    0x5D,
    0x82,
    0x34,
    0x50,
    0x5D,
    0x82,
    0x70,
    0x72,
    0x5D,
    0x82,
    0x70,
    0x0E,
    0x79,
    0x82,
    0x70,
    0x0E,
    0x84,
    0x21,
    0x70,
    0x0E,
    0x84,
    0xF3,
    0x7D,
    0x0E,
    0x84,
    0xF3,
    0x72,
    0x00,
    0x84,
    0xF3,
    0x72,
    0xE7,
]   //check里的十六进制值


for d in range(0, len(data), 5):   //for循环，每5个字符遍历一次，len：长度
    print(chr(data[d]), end="")   //将十六进制转换为ascii码然后输出，end：将这些字符整理为一行

```
执行我们写的脚本
![在这里插入图片描述](https://img-blog.csdnimg.cn/ac56f4e6c84d4c3f85b1e78875112683.png)
flag：
```
HTB{4_r4th3r_0ffb34t_b1n4ry!}
```
