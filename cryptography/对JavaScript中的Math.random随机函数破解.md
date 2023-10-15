# 什么是随机

![在这里插入图片描述](https://img-blog.csdnimg.cn/a0157334edae4ec49ea69d5705f196dc.png)


在通常的说法中，随机性是指事件中明显实际缺乏可预测性。事件、符号或步骤通常没有顺序

举个例子，比如我们在抛硬币，硬币的结果取决于很多因素，比如说我们施加的力，空气阻力，引力等，但是如果我们知道影响硬币的所有因素，并且知道这些因素准确的值，我们就能预测硬币落下时，是正面还是反面

# 伪随机

假如我们有一个算法，我们给他设置一个初始值，然后用这个算法生成下一个数字，然后再用生成的这个数字继续生成下一个……
```
12 --------> 30 --------> 57 --------> 33
```
这个初始值被称之为seed（种子），过段时间后我们可以得到一组相同的数字，这个过程叫做周期，周期越大，说明我们的算法越好，这种随机数生成的算法被称之为伪随机，它不是真正的随机生成



伪随机生成器通常用于游戏中，比如角色一开始生成的地方，但是不会用于密码学中，因为在一定条件下，伪随机生成器是可以被破解的




# Math.random
在JavaScript中，有一个叫Math.random随机生成的函数，他是一个原生的函数，几乎所有的JavaScript框架都包含这个函数
```
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/25003e407b064d4f9182aeebb2ca20d5.png)

Math.random函数会返回一个大于0且小于1的浮点伪随机数，chrome浏览器也用的是JavaScript，并且包含了这个函数，他们使用的引擎是v8




而v8是开源的，并且是由google进行维护的
```
ht#tps://github.com/v8/v8/tree/7a4a6cc6a85650ee91344d0dbd2c53a8fa8dce04
```

```
htt#ps://v8.dev/blog/math-random
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/1f647d0d7cd041cb9c6371e9c340aeae.png)



![在这里插入图片描述](https://img-blog.csdnimg.cn/713b27981f2a44d893a6109640a877d4.png)

可以发现有一个xorshift128+的算法，现在我们去看看math.random函数的源代码

![在这里插入图片描述](https://img-blog.csdnimg.cn/a0db887ac6ac41c4b0cf8891f47824cc.png)
```
static inline void XorShift128(uint64_t* state0, uint64_t* state1) {
    uint64_t s1 = *state0;
    uint64_t s0 = *state1;
    *state0 = s0;
    s1 ^= s1 << 23;
    s1 ^= s1 >> 17;
    s1 ^= s0;
    s1 ^= s0 >> 26;
    *state1 = s1;
  }
```

看起来并不复杂，只是一些异或操作和移位的操作，这个函数将state0和state1变成s0和s1，然后s1进行了异或和移位

我们不用对整个算法进行理解然后再去破解它，因为我们可以用z3这个smt求解器
# z3
![在这里插入图片描述](https://img-blog.csdnimg.cn/30269843f129420a85c16d593003c9ee.png)

z3是由微软公司开发的smt求解器，我们可以将问题作为方程式列出，然后指定约束，z3就可以自动的解决这个问题

比如下面这个方程式
```
x + 2y = 7
```
我们可以使用z3来帮助我们计算x和y的值
```
from z3 import *

x = Int('x')
y = Int('y')
solve(x + 2*y == 7) 
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/636581974d1e4cc6ada99ccb1db9e30c.png)

我们还可以用z3进行逻辑推理，弱哈希破解，数独……
# 对Math.random函数破解

![在这里插入图片描述](https://img-blog.csdnimg.cn/5f218e647dd34f61b52659f564a42600.png)


首先，我们用z3创建64位的占位符值，因为xorshift128有两个状态，所以我们将这两个状态设置为占位符变量

```
solver = z3.Solver()
se_state0, se_state1 = z3.BitVecs("se_state0 se_state1", 64)
```
然后我们还需要从v8生成的随机值进行运算，我们按下f12，进入控制台，输入以下代码，随机生成五个数
```
Array.from(Array(5), Math.random)
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0074ff7af9fd4a418ed93a5190da48a1.png)

把这五个数放到我们的脚本里，遍历这五个随机数
```
sequence = [0.6199046082820001, 0.6623637813965961, 0.7190181683749095, 0.06169296721449724, 0.915799780594273]
sequence = sequence[::-1]
```

然后我们将v8 xorshift128算法的源代码复制过来，做一些改动

![在这里插入图片描述](https://img-blog.csdnimg.cn/e753af9d4a074d3ca1c0e312644e7bca.png)

```
for i in range(len(sequence)):
    se_s1 = se_state0   #用占位符值替换状态
    se_s0 = se_state1   #用占位符值替换状态
    se_state0 = se_s0   
    se_s1 ^= se_s1 << 23
    se_s1 ^= z3.LShR(se_s1, 17)  #设置逻辑移位而不是算数移位
    se_s1 ^= se_s0
    se_s1 ^= z3.LShR(se_s0, 26)
    se_state1 = se_s1
```
然后还需要写一些代码来进行运算
```
	float_64 = struct.pack("d", sequence[i] + 1) #然后获取 se_state0的值并用struct模块把它变成双精度的值
    u_long_long_64 = struct.unpack("<Q", float_64)[0] #将其解压位64位无符号的整数
    mantissa = u_long_long_64 & ((1 << 52) - 1) #得到尾数的低52位再减去1
    solver.add(int(mantissa) == z3.LShR(se_state0, 12)) #添加约束来比较尾数
```
在v8里，还有一个函数叫做todouble，它的作用是转换，我们将这个源代码也放入我们的脚本

![在这里插入图片描述](https://img-blog.csdnimg.cn/54cc1487ba2744399253a82a6a4981a6.png)
```
	state0 = states["se_state0"].as_long()
    u_long_long_64 = (state0 >> 12) | 0x3FF0000000000000
    float_64 = struct.pack("<Q", u_long_long_64)
    next_sequence = struct.unpack("d", float_64)[0]
    next_sequence -= 1
```

现在我们就有一个可以破解Math.random函数的脚本了

脚本完全代码
```
#!/usr/bin/python3
import z3,struct,sys

sequence = [0.6199046082820001, 0.6623637813965961, 0.7190181683749095, 0.06169296721449724, 0.915799780594273]

sequence = sequence[::-1]
solver = z3.Solver()
se_state0, se_state1 = z3.BitVecs("se_state0 se_state1", 64)

for i in range(len(sequence)):
    se_s1 = se_state0
    se_s0 = se_state1
    se_state0 = se_s0
    se_s1 ^= se_s1 << 23
    se_s1 ^= z3.LShR(se_s1, 17)
    se_s1 ^= se_s0
    se_s1 ^= z3.LShR(se_s0, 26)
    se_state1 = se_s1

    float_64 = struct.pack("d", sequence[i] + 1)
    u_long_long_64 = struct.unpack("<Q", float_64)[0]
    mantissa = u_long_long_64 & ((1 << 52) - 1)
    solver.add(int(mantissa) == z3.LShR(se_state0, 12))

if solver.check() == z3.sat:
    model = solver.model()

    states = {}
    for state in model.decls():
        states[state.__str__()] = model[state]

    state0 = states["se_state0"].as_long()
    u_long_long_64 = (state0 >> 12) | 0x3FF0000000000000
    float_64 = struct.pack("<Q", u_long_long_64)
    next_sequence = struct.unpack("d", float_64)[0]
    next_sequence -= 1

    print(next_sequence)
```
运行这个脚本，获得下一个随机数

![在这里插入图片描述](https://img-blog.csdnimg.cn/fb13f25686a948a4bbc177f346fcba21.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/f89d82591d1543b493ebfed710adb2f2.png)


成功正确预测了下一个随机数

